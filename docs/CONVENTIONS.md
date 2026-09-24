# Convenções de código

## Geral
- TypeScript `strict: true`. Evite `any` (use `unknown` e faça narrowing).
- Nomes em inglês: `camelCase` para variáveis e funções, `PascalCase` para tipos e componentes, `SCREAMING_SNAKE_CASE` para constantes.
- Arquivos em `kebab-case.ts`. Componentes React em `PascalCase.tsx`.
- Funções pequenas e puras sempre que possível. Regras de negócio **não** ficam em rotas nem em componentes.
- Nada de `console.log` em código mergeado (use o logger do Fastify na API).
- Formatação: Prettier. Lint: ESLint (config na raiz). O CI reprova se falhar.

## Pacote compartilhado (`packages/shared`)
- Todo payload de request e response tem um **schema Zod** aqui. Os tipos saem de `z.infer`.
- Front e back **importam** daqui e nunca redefinem os tipos.
- Deve espelhar `docs/api/openapi.yaml`. Divergência = bug do arquiteto.
- Constantes de regra ficam aqui: `MAX_FUTURE_APPOINTMENTS = 3`, `CLIENT_CANCEL_MIN_HOURS = 2`, `BOOKING_MIN_ADVANCE_MINUTES = 60`, `BOOKING_MAX_DAYS_AHEAD = 30`.

## API: formato de respostas

### Sucesso
- Um objeto retorna o objeto diretamente.
- Uma lista paginada retorna `{ "data": [...], "meta": { "page": 1, "pageSize": 20, "total": 57 } }`.
- Uma lista sem paginação (ex.: `/services`) retorna `{ "data": [...] }`.

### Erro (sempre este formato)
```json
{
  "error": {
    "code": "SLOT_UNAVAILABLE",
    "message": "Este horário não está mais disponível.",
    "details": {}
  }
}
```
- `code`: estável, em inglês, usado pelo frontend para decidir o que fazer.
- `message`: em português, pode ser exibida ao usuário.
- `details`: opcional. Na validação, traz `{ "fields": { "email": "E-mail inválido" } }`.

### Tabela de códigos de erro

| HTTP | code | Quando |
|---|---|---|
| 400 | `BAD_REQUEST` | JSON malformado |
| 401 | `UNAUTHENTICATED` | Sem sessão ou sessão expirada |
| 401 | `INVALID_CREDENTIALS` | Login inválido |
| 403 | `FORBIDDEN` | Autenticado, mas sem permissão |
| 404 | `NOT_FOUND` | Recurso inexistente **ou de outro cliente** |
| 409 | `EMAIL_ALREADY_EXISTS` | Cadastro com e-mail repetido |
| 409 | `SERVICE_NAME_EXISTS` | Nome de serviço ativo repetido |
| 409 | `SLOT_UNAVAILABLE` | Horário ocupado, bloqueado ou fora da janela |
| 409 | `CONFLICTS_WITH_APPOINTMENTS` | Bloqueio sobrepõe agendamentos (HB-02) |
| 409 | `INVALID_STATUS_TRANSITION` | Transição proibida (AG-08/09/10) |
| 422 | `VALIDATION_ERROR` | Falha de validação de campos |
| 422 | `SERVICE_INACTIVE` | Agendar um serviço desativado |
| 422 | `BOOKING_TOO_SOON` | Menos de 1h de antecedência |
| 422 | `BOOKING_TOO_FAR` | Mais de 30 dias à frente |
| 422 | `MAX_APPOINTMENTS_REACHED` | Cliente com 3 agendamentos futuros |
| 422 | `CANCELLATION_WINDOW_EXPIRED` | Cliente cancelando com menos de 2h |
| 429 | `RATE_LIMITED` | Muitas tentativas |
| 500 | `INTERNAL_ERROR` | Erro inesperado (sem detalhes internos) |

> Recurso de outro cliente retorna **404**, e não 403, para não revelar que o recurso existe.

## API: rotas
- Prefixo `/api`. Recursos no plural: `/services`, `/appointments`.
- Ações que não são CRUD usam sub-recurso com verbo: `POST /appointments/{id}/cancel`.
- Datas sempre em ISO 8601 UTC. Dinheiro sempre em centavos (`priceCents`).

## Frontend
- Chamadas HTTP só via `src/lib/api/` (cliente tipado com schemas do shared), sempre com **URL relativa** `/api/...`.
- Estado de servidor com TanStack Query. Nada de `useEffect` + `fetch` manual.
- Toda tela que busca dados trata **4 estados**: carregando (skeleton), vazio, erro (com "tentar novamente") e sucesso.
- Mapeie o `error.code` para mensagens amigáveis em `src/lib/api/error-messages.ts`.
- Formate dinheiro com `Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' })`.
- Formate datas com `date-fns-tz` no fuso `America/Sao_Paulo`.
- Acessibilidade: todo input tem `<label>`, botões com texto ou `aria-label`, foco visível, contraste AA.
- Mobile first: projete para 360 px de largura e depois amplie.

## Banco (Prisma)
- Modelos em `PascalCase` singular. Tabelas mapeadas para `snake_case` plural (`@@map("appointments")`).
- Colunas mapeadas para `snake_case` (`@map("starts_at")`).
- Toda FK tem índice. Datas usam o tipo `timestamptz` (`@db.Timestamptz(3)`).
- Migrações **nunca** são editadas depois do merge. Mudou? Crie uma migração nova.
- SQL cru (ex.: exclusion constraint) fica dentro da migração, com comentário explicando.

## Testes
| Tipo | Ferramenta | Onde | O que cobrir |
|---|---|---|---|
| Unitário | Vitest | ao lado do arquivo (`*.test.ts`) | Funções puras, principalmente o cálculo de disponibilidade e as transições de status |
| Integração API | Vitest + `fastify.inject` + Postgres de teste | `apps/api/src/**/*.int.test.ts` | Cada rota: sucesso, validação, auth, regra de negócio |
| Componente | Vitest + Testing Library | `apps/web/src/**/*.test.tsx` | Formulários e estados de tela |
| E2E | Playwright | `e2e/` | Fluxos críticos completos |

- Nome do teste descreve o comportamento: `it('retorna 409 quando o horário já está ocupado')`.
- Para testar tempo, use um relógio injetável (`now()` como parâmetro ou `vi.useFakeTimers`). Nunca dependa da hora real.
- Cobertura mínima: 80% nos `service.ts` e em `availability/`.

## Commits
Conventional Commits com escopo: `api`, `web`, `db`, `shared`, `contrato`, `ci`, `e2e`, `docs`.
```
feat(api): cria POST /appointments
fix(web): corrige fuso na lista de horários
test(api): cobre cancelamento fora da janela
```
