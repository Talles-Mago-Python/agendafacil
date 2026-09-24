# ADR 004: Datas em UTC e fuso único do negócio

- **Status:** aceito
- **Data:** 2026-09-24

## Contexto
Bugs de fuso horário são a causa nº 1 de erros em sistemas de agenda. O cliente pode estar em outro fuso (viajando), e o servidor roda em UTC.

## Decisão
1. **O banco guarda tudo em UTC** (`timestamptz`), e **a API trafega tudo em ISO 8601 UTC** com `Z`.
2. Existe **um único fuso de negócio**, dado por `BUSINESS_TIMEZONE` (padrão `America/Sao_Paulo`).
3. `BusinessHours.opensAt/closesAt` (`"HH:mm"`) e o parâmetro `date` (`YYYY-MM-DD`) de `/availability` são interpretados **no fuso do negócio**.
4. O frontend **exibe** e **coleta** datas no fuso do negócio (e não no do navegador), com `date-fns-tz` (`formatInTimeZone`, `fromZonedTime`).
5. Conversões ficam em funções utilitárias únicas: `apps/api/src/lib/dates.ts` e `apps/web/src/lib/dates.ts`, com testes.

## Consequências
- ✅ "09:00" significa sempre 09:00 no relógio da barbearia.
- ✅ Funciona mesmo se o Brasil voltar a ter horário de verão (o `date-fns-tz` usa a base IANA).
- ⚠️ Proibido usar `new Date('2026-10-06')` (interpretado como UTC) ou `getHours()` (fuso da máquina) em regra de negócio.
- ⚠️ Os testes devem rodar com `TZ=UTC` e também com `TZ=America/Sao_Paulo`, e o resultado não pode mudar. O CI roda com `TZ=UTC`.

## Alternativas consideradas
| Alternativa | Por que foi descartada |
|---|---|
| Exibir no fuso do navegador | Um cliente viajando veria horários "errados" em relação ao estabelecimento |
| Guardar horário local sem fuso | Ambíguo e quebra comparações |
| Temporal API | Ainda sem suporte amplo e estável em todos os ambientes alvo |
