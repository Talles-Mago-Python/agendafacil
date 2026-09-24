# AgendaFácil

Sistema web de agendamento online para pequenos negócios.

> Projeto desenvolvido com **vibecoding multiagente** (Arena.ai + GitHub).
> Agentes de IA: leiam primeiro o [`AGENTS.md`](./AGENTS.md).

## Documentação

| Documento | Conteúdo |
|---|---|
| [`docs/VISION.md`](docs/VISION.md) | O que é o produto, para quem, escopo do MVP |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Stack, estrutura, fluxo de dados, variáveis de ambiente |
| [`docs/DOMAIN.md`](docs/DOMAIN.md) | Entidades e regras de negócio |
| [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md) | Padrões de código, erros, testes |
| [`docs/api/openapi.yaml`](docs/api/openapi.yaml) | Contrato da API |
| [`docs/specs/`](docs/specs/) | Especificação de cada funcionalidade |
| [`docs/adr/`](docs/adr/) | Decisões de arquitetura |
| [`docs/BACKLOG.md`](docs/BACKLOG.md) | Issues planejadas por fase |
| [`docs/PROGRESS.md`](docs/PROGRESS.md) | Estado atual do projeto |
| [`docs/HANDOFF.md`](docs/HANDOFF.md) | Pedidos entre agentes |

## Rodando localmente

Pré-requisitos: Node 24+, pnpm 9+, Docker.

```bash
pnpm install
cp .env.example .env
docker compose up -d db
pnpm db:migrate && pnpm db:seed
pnpm dev
```

- Web: http://localhost:3000
- API: http://localhost:3333/api/health

Usuários do seed:
- Admin: `admin@agendafacil.dev` / `Admin@123`
- Cliente: `cliente@agendafacil.dev` / `Cliente@123`
