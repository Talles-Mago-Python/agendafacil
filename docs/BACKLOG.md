# Backlog do MVP

> O agente **arquiteto** transforma cada linha em uma issue no GitHub, usando o template `tarefa`.
> Formato: `ID | título da issue | área | tamanho | depende de | spec`
> Tamanho: **P** ≈ até 2h de agente · **M** ≈ meio dia. Nada maior que M: se for maior, quebre.

## Fase 1: Fundação

| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F1-01 | Esqueleto do monorepo (pnpm workspaces, tsconfig base, ESLint, Prettier) | devops | M | – | ARCHITECTURE §3, ADR-001 |
| F1-02 | docker-compose com Postgres 16 + `.env.example` | devops | P | F1-01 | ARCHITECTURE §6 |
| F1-03 | Workflow de CI: install, lint, test, build em todo PR (TZ=UTC, Postgres service) | devops | M | F1-01 | ADR-004 |
| F1-04 | `packages/shared`: schemas Zod + tipos espelhando o openapi.yaml v0.1 | arquiteto | M | F1-01 | openapi.yaml, CONVENTIONS |
| F1-05 | `packages/shared`: constantes de regra e enum de códigos de erro | arquiteto | P | F1-04 | DOMAIN, CONVENTIONS |
| F1-06 | Esqueleto `apps/api`: Fastify, `/api/health`, plugin de erros, config por env | backend | M | F1-01 | ARCHITECTURE §4 |
| F1-07 | Esqueleto `apps/web`: Next + Tailwind + TanStack Query + rewrite `/api/*` + layout base | frontend | M | F1-01 | ADR-003 |
| F1-08 | Proteger branch `main` + labels + templates (humano) | humano | P | – | – |

**Marco da fase:** `pnpm install && pnpm build` OK, CI verde, `/api/health` responde pelo rewrite do Next.

## Fase 2: Construção

### Dados
| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F2-01 | Schema Prisma: User, Service, BusinessHours, BlockedSlot, Appointment + migração inicial | dados | M | F1-02 | DOMAIN §2, CONVENTIONS/Banco |
| F2-02 | Migração com exclusion constraint `appointments_no_overlap` | dados | P | F2-01 | ADR-005 |
| F2-03 | Seed: admin, cliente, 5 serviços, 7 dias de horário, 3 agendamentos, 1 bloqueio | dados | P | F2-01 | README (usuários) |

### Backend
| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F2-10 | Auth: register, login, logout, me + plugin de sessão + rate limit | backend | M | F2-01, F1-06 | specs/auth.md, ADR-003 |
| F2-11 | Guards `requireAuth` / `requireAdmin` | backend | P | F2-10 | specs/auth.md |
| F2-12 | CRUD de serviços | backend | M | F2-11 | specs/servicos.md |
| F2-13 | Business hours GET/PUT | backend | P | F2-11 | specs/horarios-e-bloqueios.md |
| F2-14 | Blocked slots GET/POST/DELETE com checagem de conflito | backend | M | F2-13 | specs/horarios-e-bloqueios.md |
| F2-15 | Função pura `calculateAvailableSlots` + 8 testes obrigatórios | backend | M | F1-05 | specs/agendamento.md §1 |
| F2-16 | GET `/availability` | backend | P | F2-15, F2-13 | specs/agendamento.md §1 |
| F2-17 | POST `/appointments` (validações + mapeamento 23P01 → 409 + teste de concorrência) | backend | M | F2-16, F2-02 | specs/agendamento.md §2, ADR-005 |
| F2-18 | GET `/appointments` (lista paginada) e GET `/appointments/{id}` | backend | M | F2-17 | specs/agendamento.md §3 |
| F2-19 | Cancelar + atualizar status | backend | M | F2-18 | specs/agendamento.md §4-5 |

### Frontend (usa mock MSW até o backend existir)
| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F2-30 | Cliente HTTP tipado + mapeamento de erros + handlers MSW do contrato | frontend | M | F1-04, F1-07 | CONVENTIONS/Frontend |
| F2-31 | Utilitários de data/moeda no fuso do negócio + testes | frontend | P | F1-07 | ADR-004 |
| F2-32 | Telas de login e cadastro + contexto de sessão + proteção de rotas | frontend | M | F2-30 | specs/auth.md |
| F2-33 | Home + lista pública de serviços | frontend | P | F2-30 | specs/servicos.md |
| F2-34 | Fluxo de agendamento (dias → horários → confirmação → sucesso) | frontend | M | F2-31, F2-33 | specs/agendamento.md (Telas) |
| F2-35 | Meus agendamentos (próximos/histórico + cancelar) | frontend | M | F2-34 | specs/agendamento.md (Telas) |
| F2-36 | Admin: agenda dia/semana + painel de detalhes + ações | frontend | M | F2-31 | specs/agendamento.md (Telas) |
| F2-37 | Admin: CRUD de serviços | frontend | M | F2-30 | specs/servicos.md |
| F2-38 | Admin: horários de funcionamento + bloqueios | frontend | M | F2-31 | specs/horarios-e-bloqueios.md |

## Fase 3: Integração e qualidade

| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F3-01 | Desligar MSW por padrão (ligar via `NEXT_PUBLIC_API_MOCKING=true`) e testar contra a API real | frontend | P | Fase 2 | – |
| F3-02 | Setup Playwright + E2E: cadastro → agendar → ver em "meus agendamentos" | qa | M | F3-01 | specs/agendamento.md |
| F3-03 | E2E: admin cria serviço, bloqueia horário e marca atendimento como concluído | qa | M | F3-02 | specs/* |
| F3-04 | E2E no CI (workflow separado) | devops | P | F3-02 | – |
| F3-05 | Revisão de segurança (checklist ARCHITECTURE §8) | qa | P | Fase 2 | ARCHITECTURE §8 |
| F3-06 | Auditoria Lighthouse mobile ≥ 90 + correções | frontend | M | F3-01 | VISION (métricas) |

## Fase 4: Deploy

| ID | Título | Área | Tam. | Depende | Referência |
|---|---|---|---|---|---|
| F4-01 | Dockerfile da API + deploy no Render com Postgres gerenciado | devops | M | Fase 3 | ARCHITECTURE §7 |
| F4-02 | `prisma migrate deploy` no deploy da API | devops | P | F4-01 | – |
| F4-03 | Web na Vercel com `API_URL` apontando para a API | devops | P | F4-01 | ADR-003 |
| F4-04 | Documentar os secrets necessários e o processo de deploy no README | devops | P | F4-03 | – |
| F4-05 | Smoke test pós-deploy (`/api/health` + home) | devops | P | F4-03 | – |

## Ideias para v2 (não criar issues ainda)
Vários profissionais · lembretes por e-mail/WhatsApp · recuperação de senha · pagamento de sinal · editar perfil · relatórios de faturamento.
