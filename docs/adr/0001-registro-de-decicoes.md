# 0001 — Registrar decisões de arquitetura em ADRs

- **Data:** 2026-09-24
- **Status:** ACEITA
- **Decisores:** Agente Arquiteto (proposta), pendente de confirmação do humano

## Contexto
O repositório começa do zero e vários agentes (dados, backend, frontend, devops, QA) vão
trabalhar em paralelo sobre o mesmo contrato. Sem registro, decisões implícitas viram retrabalho
e divergência entre áreas.

## Decisão
Toda decisão estrutural (stack, modelo de dados, tenancy, política de fuso horário, estratégia
de erro, etc.) é registrada como ADR numerado em `docs/adr/`, usando `_TEMPLATE.md`.
Uma decisão só é considerada válida se houver ADR em status **ACEITA**.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|---|---|---|---|
| Decisões só em chat/PR | Rápido | Não pesquisável, some com o histórico | Agentes não têm memória entre sessões |
| README longo | Simples | Mistura uso com decisão, apodrece | Dificulta saber o que está vigente |

## Consequências
- Custo pequeno por decisão (um arquivo curto) em troca de rastreabilidade.
- Mudar de ideia é permitido: cria-se ADR novo que marca o anterior como SUPERSEDA —
  nunca se edita um ADR aceito para apagar a decisão antiga.
- `docs/ARCHITECTURE.md` passa a apontar para os ADRs vigentes em vez de duplicá-los.
