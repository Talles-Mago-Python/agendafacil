# ADR 005: Impedir sobreposição de agendamentos no banco (exclusion constraint)

- **Status:** aceito
- **Data:** 2026-09-24

## Contexto
A regra AG-03 diz que nunca podem existir dois agendamentos `CONFIRMED` sobrepostos. Uma checagem só na aplicação ("SELECT e depois INSERT") falha quando duas requisições chegam ao mesmo tempo (*race condition*).

## Decisão
Usar uma **exclusion constraint** do PostgreSQL com a extensão `btree_gist`, criada em SQL cru dentro de uma migração do Prisma:

```sql
-- Garante AG-03: nenhum par de agendamentos CONFIRMED com intervalos sobrepostos.
-- '[)' = intervalo semiaberto: 09:00-10:00 não conflita com 10:00-11:00 (DS-03).
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE appointments
  ADD CONSTRAINT appointments_no_overlap
  EXCLUDE USING gist (
    tstzrange(starts_at, ends_at, '[)') WITH &&
  ) WHERE (status = 'CONFIRMED');
```

A API continua checando a disponibilidade antes (para devolver mensagens boas), mas **o banco é a garantia final**. O backend captura o erro do Postgres `23P01` (exclusion_violation) e o converte em `409 SLOT_UNAVAILABLE`.

## Consequências
- ✅ Impossível ter agendamentos duplicados, mesmo com bugs ou concorrência.
- ✅ Na v2 (vários profissionais), basta adicionar `professional_id WITH =` à constraint.
- ⚠️ O Prisma não conhece a constraint. O agente dados precisa criar a migração com `prisma migrate dev --create-only` e editar o SQL.
- ⚠️ Os testes de integração precisam de um Postgres real (não usar SQLite).
- ⚠️ O agente backend precisa mapear o código `23P01` (vem em `PrismaClientKnownRequestError` / `meta`) para 409.

## Alternativas consideradas
| Alternativa | Por que foi descartada |
|---|---|
| Só checagem na aplicação | Falha em concorrência |
| `SELECT ... FOR UPDATE` / lock da tabela | Complexo, fácil de errar e prejudica a performance |
| Unique em `(starts_at)` | Não pega sobreposição parcial (09:00–10:00 vs 09:30–10:30) |
| Isolamento SERIALIZABLE | Exige retry na aplicação e é mais difícil de testar |
