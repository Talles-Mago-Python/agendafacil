# Arquitetura: AgendaFácil

## 1. Visão geral

```
                Navegador (cliente / admin)
                           │  HTTPS, mesma origem
                           ▼
        ┌─────────────────────────────────────┐
        │  apps/web: Next.js (App Router)     │
        │  - páginas e componentes            │
        │  - rewrite: /api/*  →  API_URL/api/*│
        └─────────────────────────────────────┘
                           │  HTTP (servidor → servidor)
                           ▼
        ┌─────────────────────────────────────┐
        │  apps/api: Fastify + TypeScript     │
        │  - rotas /api/*                     │
        │  - validação (Zod via shared)       │
        │  - auth por cookie httpOnly (JWT)   │
        └─────────────────────────────────────┘
                           │  Prisma Client
                           ▼
        ┌─────────────────────────────────────┐
        │  PostgreSQL 16                      │
        └─────────────────────────────────────┘
```

**Por que o rewrite?** O navegador só conversa com o domínio do site, e o Next repassa `/api/*` para a API. Com isso, **o cookie de sessão é first-party** e não há CORS (ADR-003). O frontend **sempre** usa URLs relativas (`/api/...`).

## 2. Stack

| Camada | Tecnologia | Versão alvo |
|---|---|---|
| Linguagem | TypeScript (strict) | 5.x |
| Runtime | Node.js | 20 LTS |
| Gerenciador | pnpm workspaces | 9.x |
| Frontend | Next.js (App Router) + React | 15.x / 19.x |
| Estilo | Tailwind CSS | 4.x |
| Dados no front | TanStack Query | 5.x |
| Formulários | React Hook Form + Zod | – |
| Backend | Fastify | 5.x |
| Validação | Zod (schemas em `packages/shared`) | 3.x |
| ORM | Prisma | 6.x |
| Banco | PostgreSQL | 16 |
| Auth | JWT (jose) em cookie httpOnly + argon2 | – |
| Datas | date-fns + date-fns-tz | – |
| Testes unitários | Vitest | – |
| Mock de API (front) | MSW | 2.x |
| E2E | Playwright | – |
| CI | GitHub Actions | – |
| Deploy | Vercel (web), Render (API + Postgres) | – |

Decisões registradas em `docs/adr/`.

## 3. Estrutura do repositório

```
agendafacil/
├── apps/
│   ├── web/                     # [frontend]
│   │   ├── src/app/             # rotas (App Router)
│   │   │   ├── (public)/        # home, serviços, login, cadastro
│   │   │   ├── (client)/        # meus agendamentos, agendar
│   │   │   └── admin/           # painel admin
│   │   ├── src/components/      # componentes reutilizáveis
│   │   ├── src/lib/api/         # cliente HTTP tipado (usa shared)
│   │   └── src/mocks/           # handlers MSW baseados no contrato
│   └── api/                     # [backend]
│       ├── src/modules/         # auth/, services/, availability/,
│       │                        # appointments/, blocked-slots/, business-hours/
│       │   └── <modulo>/        #   routes.ts, service.ts, repository.ts, *.test.ts
│       ├── src/plugins/         # auth, error-handler, prisma
│       ├── src/lib/             # datas, erros, config
│       └── src/server.ts
├── packages/
│   └── shared/                  # [arquiteto] schemas Zod + tipos + constantes
│       └── src/{schemas,types,constants,errors}.ts
├── prisma/                      # [dados] schema.prisma, migrations/, seed.ts
├── e2e/                         # [qa] testes Playwright
├── docs/                        # [arquiteto]
├── .github/                     # workflows [devops], templates [arquiteto]
├── docker-compose.yml           # [devops]
├── .env.example                 # [devops]
└── AGENTS.md
```

## 4. Camadas do backend

```
routes.ts      → HTTP: parse/validação (Zod), status code, auth guard
service.ts     → regras de negócio (DOMAIN.md), sem conhecer HTTP
repository.ts  → acesso ao banco via Prisma, sem regra de negócio
```
- O `service` lança erros de domínio (`AppError` com `code`). O plugin `error-handler` converte para o formato do contrato.
- As regras de disponibilidade ficam em **funções puras** (`availability/calculate.ts`), fáceis de testar.

## 5. Módulos e responsabilidades

| Módulo | Rotas | Spec |
|---|---|---|
| auth | `/auth/*` | `specs/auth.md` |
| services | `/services*` | `specs/servicos.md` |
| business-hours | `/business-hours` | `specs/horarios-e-bloqueios.md` |
| blocked-slots | `/blocked-slots*` | `specs/horarios-e-bloqueios.md` |
| availability | `/availability` | `specs/agendamento.md` |
| appointments | `/appointments*` | `specs/agendamento.md` |

## 6. Variáveis de ambiente

| Variável | Onde | Exemplo | Descrição |
|---|---|---|---|
| `DATABASE_URL` | api, prisma | `postgresql://postgres:postgres@localhost:5432/agendafacil` | Conexão com o Postgres |
| `JWT_SECRET` | api | (64+ chars aleatórios) | Assinatura do token de sessão |
| `SESSION_TTL_DAYS` | api | `7` | Validade da sessão |
| `BUSINESS_TIMEZONE` | api | `America/Sao_Paulo` | Fuso do negócio (ADR-004) |
| `SLOT_INTERVAL_MINUTES` | api | `15` | Granularidade dos horários ofertados |
| `API_PORT` | api | `3333` | Porta da API |
| `NODE_ENV` | api, web | `development` | Ambiente |
| `API_URL` | web (servidor) | `http://localhost:3333` | Destino do rewrite `/api/*` |

Variáveis **nunca** vão para o código. Os valores de produção ficam nos provedores e em *GitHub → Settings → Secrets*.

## 7. Ambientes

| Ambiente | Web | API | Banco |
|---|---|---|---|
| local | `localhost:3000` | `localhost:3333` | Docker Compose |
| preview (por PR) | Vercel Preview | API de staging | Postgres de staging |
| produção | Vercel | Render | Render Postgres |

## 8. Segurança (resumo)
- Senhas com **argon2id**. Nunca logar senha, hash ou token.
- Cookie de sessão `af_session`: `HttpOnly`, `Secure` (em produção), `SameSite=Lax`, `Path=/`.
- Autorização checada **no backend** em toda rota (o frontend só esconde botões).
- Rate limit em `/auth/login` e `/auth/register` (5 tentativas/min por IP).
- Toda entrada é validada com Zod, e o Prisma impede SQL injection.
- Headers de segurança via `@fastify/helmet`.
