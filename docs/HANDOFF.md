# Handoff: pedidos entre agentes

> Use quando precisar de algo **fora da sua área**. Registre o pedido, avise o humano e **pare** (não implemente na área do outro).
> O arquiteto (ou o agente dono) responde e move o item para "Resolvidos".

## Como registrar

```markdown
### H-NNN · [de: área] → [para: área] · Título curto
- **Data:** AAAA-MM-DD
- **Issue relacionada:** #N
- **Preciso de:** o que exatamente (campo, endpoint, variável de ambiente, dependência...)
- **Motivo:** por que é necessário (link para a spec, se houver)
- **Bloqueia minha tarefa?** sim / não (sigo com um mock ou contorno)
- **Status:** aberto
```

## Exemplos de pedidos válidos
- O frontend precisa de um campo novo na resposta → **para: arquiteto** (muda o contrato)
- O backend precisa de um índice novo → **para: dados**
- Qualquer agente precisa de uma variável de ambiente nova ou de uma dependência na raiz → **para: devops**
- Uma regra de negócio ambígua ou contraditória → **para: arquiteto** (esclarece no DOMAIN.md)

---

## Abertos

### H-001 · [de: dados] → [para: devops] · Prisma 6.19.3 fixo e setup do banco na raiz
- **Data:** 2026-09-24
- **Issue relacionada:** ainda não existe. Afeta F1-01 e F1-02, que são pré-requisito de F2-01 a F2-03.
- **Preciso de:**
  1. `prisma` e `@prisma/client` na **mesma versão exata, `6.19.3`**, no `package.json` da raiz, porque o `prisma/` fica na raiz (ADR-001). O backend usa a mesma versão de `@prisma/client` em `apps/api`. ⚠️ **Não instale pela tag `latest`:** hoje `prisma@latest` é `8.0.0-rc.15`, um RC do Prisma ORM 8 com formato de schema diferente, e `@prisma/client@latest` é `7.10.0`. O ARCHITECTURE pede Prisma 6.x.
  2. `"prisma": { "schema": "prisma/schema.prisma", "seed": "tsx prisma/seed.ts" }` no `package.json` da raiz (ou o equivalente em `prisma.config.ts`), com `tsx` em devDependencies.
  3. Scripts na raiz com os nomes usados no README: `db:migrate` → `prisma migrate dev` e `db:seed` → `prisma db seed`. Sugestão: também `db:reset` → `prisma migrate reset` e `db:generate` → `prisma generate`.
  4. `DATABASE_URL` no `.env.example` (ARCHITECTURE §6). No docker-compose (F1-02) e no service do CI (F1-03), um Postgres 16 cujo usuário possa rodar `CREATE EXTENSION btree_gist`, que a migração da ADR-005 executa. O usuário `postgres` da imagem oficial já pode.
- **Motivo:** o agente de dados só edita `prisma/`, e pelo ADR-001 só o devops altera a raiz. Sem esses itens não dá para rodar `migrate dev` nem o seed de F2-01 a F2-03 (DOMAIN §2, CONVENTIONS/Banco, ADR-005).
- **Bloqueia minha tarefa?** sim. F2-01 depende de F1-02.
- **Status:** aberto

## Resolvidos
_(nenhum)_
