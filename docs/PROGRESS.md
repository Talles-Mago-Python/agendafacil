# Progresso

> Diário de bordo do projeto. **Todo agente atualiza ao iniciar e ao concluir uma tarefa.**
> Formato: `AAAA-MM-DD [área] #issue descrição (branch ou PR)`

## Estado atual
- **Fase:** 1, Fundação
- **Versão do contrato:** 0.1.0 (`docs/api/openapi.yaml`)
- **Bloqueios:** nenhum

## Em andamento
_(nenhuma tarefa em andamento)_

## Próximos (em ordem)
1. [humano] F1-08 Proteger a `main`, criar labels, conectar o repositório ao Arena.ai
2. [arquiteto] Criar as issues da Fase 1 a partir do `docs/BACKLOG.md`
3. [devops] F1-01 Esqueleto do monorepo
4. [devops] F1-02 / F1-03 docker-compose e CI
5. [arquiteto] F1-04 / F1-05 `packages/shared`
6. [backend] F1-06 · [frontend] F1-07 (em paralelo)

## Concluído
- 2026-09-24 [arquiteto] Documentação inicial: VISION, ARCHITECTURE, DOMAIN, CONVENTIONS, specs (auth, serviços, horários e bloqueios, agendamento), openapi.yaml v0.1.0, ADRs 001–005, BACKLOG
- 2026-09-24 [dados] Documentos enviados pelo humano organizados em `docs/`. Removidas as sobras do bootstrap anterior: ADR 0001 e templates duplicados. Ainda faltam no repositório o `AGENTS.md` e o `openapi.yaml` v0.1.0. Aberto o H-001 para o devops (arena/01a0d47e-agendafacil)
- 2026-09-24 [dados] A pedido do humano: removido `docs/specs/_TEMPLATE.md` e Node alvo trocado de 20 para 24 LTS no ARCHITECTURE e no README (arena/01a0d47e-agendafacil)

## Decisões recentes
- Veja `docs/adr/` (001 a 005)
- 2026-09-24 · Node alvo: **24 LTS** (decisão do humano). O Node 20 saiu de suporte em 30/04/2026. O 26 só vira LTS em 28/10/2026 e ainda não está disponível na Vercel para builds e Functions, onde o máximo hoje é o 24.x. Reavaliar quando a Vercel suportar o 26.
