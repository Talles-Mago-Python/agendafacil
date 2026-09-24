# PROGRESS.md

Painel de progresso. Atualizado pelo Agente Arquiteto a cada spec/issue concluída.

**Legenda:** ⬜ não iniciado · 🟨 em andamento · ✅ concluído · ⛔ bloqueado

## Fundações

| Item | Área | Status | Obs. |
|---|---|---|---|
| Esqueleto de processo (`AGENTS.md`, `docs/`, templates) | arquiteto | ✅ | 2026-09-24 |
| `docs/VISION.md` | arquiteto | ⛔ | Bloqueado em H-001 (perguntas de negócio) |
| ADR 0002–0006 (domínio, tenancy, operador, disponibilidade, stack) | arquiteto | ⛔ | Bloqueado em H-001 |
| Labels `area:*` no GitHub | arquiteto | ⬜ | H-002 |
| `docs/api/openapi.yaml` com rotas reais | arquiteto | ⛔ | Stub v0.0.0 criado; conteúdo depende da spec |
| `packages/shared` | arquiteto | ⛔ | Depende da stack (ADR 0006) |
| `apps/api` | backend | ⬜ | — |
| `apps/web` | frontend | ⬜ | — |
| Migrations/seeds | dados | ⬜ | — |
| CI | devops | ⬜ | — |
| Plano de testes | qa | ⬜ | — |

## Features

| # | Feature | Spec | ADRs | Issues | Status |
|---|---|---|---|---|---|
| — | _nenhuma ainda_ | — | — | — | — |

## Registro

- **2026-09-24** — Repositório recebido vazio (commit base `02fad20`, só `README.md`).
  Criados: `AGENTS.md`, `docs/ARCHITECTURE.md`, `docs/PROGRESS.md`, `docs/HANDOFF.md`,
  `docs/specs/_TEMPLATE.md`, `docs/adr/_TEMPLATE.md`, `docs/adr/0001-registro-de-decicoes.md`,
  `docs/api/openapi.yaml` (stub), `.github/ISSUE_TEMPLATE/feature.md`.
  Enviadas 5 perguntas de negócio ao humano (H-001).
