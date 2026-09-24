# Progresso

> Diário de bordo do projeto. **Todo agente atualiza ao iniciar e ao concluir uma tarefa.**
> Formato: `AAAA-MM-DD [área] #issue descrição (branch ou PR)`

## Estado atual
- **Fase:** 1, Fundação
- **Versão do contrato:** ⚠️ o `docs/api/openapi.yaml` no repositório ainda é o **stub `0.0.0`** (sem `paths`/`schemas`). A 0.1.0 anunciada aqui ainda não foi publicada — verificado em 2026-09-24.
- **Bloqueios:**
  - #4 (F1-04) precisa que o **arquiteto** publique o `openapi.yaml` v0.1.0 antes de escrever os schemas Zod.
  - Labels não podem ser aplicadas pelo token do agente (403) — pendência do F1-08, ver "Próximos".
  - `AGENTS.md` ainda não existe no repositório (o template `tarefa` e o README apontam para ele).

## Em andamento
_(nenhuma tarefa em andamento)_

## Próximos (em ordem)
1. [humano] F1-08 Proteger a `main`, criar labels, conectar o repositório ao Arena.ai
   - As 11 labels **já existem** no repositório (`area:arquiteto|dados|backend|frontend|devops|qa`, `contrato`, `bloqueada`, `tamanho:P|M`, `bug`).
   - ⚠️ **Falta aplicá-las nas issues:** o token do agente recebe `403 Resource not accessible by integration` em POST/PUT/GraphQL de labels (testado em 2026-09-24). Aplicar manualmente:
     | Issue | Labels |
     |---|---|
     | #1 | `area:devops`, `tamanho:M` |
     | #2 | `area:devops`, `tamanho:P` |
     | #3 | `area:devops`, `tamanho:M` |
     | #4 | `area:arquiteto`, `contrato`, `tamanho:M` |
     | #5 | `area:arquiteto`, `contrato`, `tamanho:P` |
     | #6 | `area:backend`, `tamanho:M` |
     | #7 | `area:frontend`, `tamanho:M` |
2. [arquiteto] Publicar o `docs/api/openapi.yaml` **v0.1.0** (hoje é o stub `0.0.0`) e escrever o `AGENTS.md` — desbloqueia #4
3. [devops] #1 (F1-01) Esqueleto do monorepo — nada depende de nada antes dele
4. [devops] #2 (F1-02) docker-compose + `.env.example` · #3 (F1-03) CI — ambos dependem de #1; o #2 resolve o item 4 do H-001
5. [arquiteto] #4 (F1-04) schemas Zod do `packages/shared` → depois #5 (F1-05) constantes e códigos de erro
6. [backend] #6 (F1-06) `apps/api` · [frontend] #7 (F1-07) `apps/web` — em paralelo, ambos dependem só de #1

## Concluído
- 2026-09-24 [arquiteto] Issues da Fase 1 criadas a partir do `docs/BACKLOG.md`: #2 (F1-02), #3 (F1-03), #4 (F1-04), #5 (F1-05), #6 (F1-06) e #7 (F1-07), todas com o template `tarefa`, ID do backlog no corpo e dependências apontando para os números reais (#1 e #4). A F1-01 já existia como #1 e a F1-08 é do humano. Labels **não** puderam ser aplicadas pelo token (403) — ver "Próximos" item 1 (arena/01a0d47e-agendafacil)
- 2026-09-24 [arquiteto] Documentação inicial: VISION, ARCHITECTURE, DOMAIN, CONVENTIONS, specs (auth, serviços, horários e bloqueios, agendamento), openapi.yaml v0.1.0, ADRs 001–005, BACKLOG
  - ⚠️ Correção de 2026-09-24: o `docs/api/openapi.yaml` **não** está na v0.1.0 — o arquivo no repositório é o stub `version: 0.0.0`, sem `paths` e sem `schemas`. Publicado como pendência em "Próximos" item 2.
- 2026-09-24 [dados] Documentos enviados pelo humano organizados em `docs/`. Removidas as sobras do bootstrap anterior: ADR 0001 e templates duplicados. Ainda faltam no repositório o `AGENTS.md` e o `openapi.yaml` v0.1.0. Aberto o H-001 para o devops (arena/01a0d47e-agendafacil)
- 2026-09-24 [dados] A pedido do humano: removido `docs/specs/_TEMPLATE.md` e Node alvo trocado de 20 para 24 LTS no ARCHITECTURE e no README (arena/01a0d47e-agendafacil)

## Decisões recentes
- Veja `docs/adr/` (001 a 005)
- 2026-09-24 · Node alvo: **24 LTS** (decisão do humano). O Node 20 saiu de suporte em 30/04/2026. O 26 só vira LTS em 28/10/2026 e ainda não está disponível na Vercel para builds e Functions, onde o máximo hoje é o 24.x. Reavaliar quando a Vercel suportar o 26.
