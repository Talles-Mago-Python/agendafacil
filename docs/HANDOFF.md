# HANDOFF.md

Fila de pedidos **para o Agente Arquiteto**. Qualquer agente (ou humano) registra aqui o que
precisa; o Arquiteto resolve e marca o item.

Formato de cada pedido:

```
### H-NNN — <título curto>
- De: <agente ou pessoa>
- Data: <AAAA-MM-DD>
- Pedido: <o que precisa>
- Bloqueia: <issue/área>
- Status: ABERTO | EM ANDAMENTO | RESOLVIDO
- Resolução: <link para spec/ADR/issue + resumo>
```

---

## Fila

### H-001 — Definir domínio, tenancy, operador, disponibilidade e stack
- De: Agente Arquiteto (autoaberto ao constatar repositório vazio)
- Data: 2026-09-24
- Pedido: responder as 5 perguntas de negócio enviadas ao humano para que `docs/VISION.md`,
  os ADRs 0002–0006 e a primeira spec possam ser escritos.
- Bloqueia: **tudo** — `docs/VISION.md`, `docs/api/openapi.yaml`, `packages/shared`, `apps/*`,
  todas as issues de área.
- Status: **ABERTO** (aguardando resposta do humano)
- Resolução: —

### H-002 — Criar labels `area:*` no GitHub
- De: Agente Arquiteto
- Data: 2026-09-24
- Pedido: `gh label create` para `area:dados`, `area:backend`, `area:frontend`, `area:devops`,
  `area:qa` (+ `spec:*`, `blocked`, `ready`). Verificado em 24/09/2026: só existem os labels
  default do GitHub.
- Bloqueia: abertura das primeiras issues com classificação correta.
- Status: **ABERTO**
- Resolução: —

---

## Resolvidos

_(vazio)_
