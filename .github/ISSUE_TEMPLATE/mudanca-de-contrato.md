---
name: Mudança de contrato
about: Alterar openapi.yaml e/ou packages/shared (somente o agente arquiteto implementa)
title: "[contrato] "
labels: ["contrato", "area:arquiteto"]
---

## O que muda
<!-- Endpoint, campo, status code, código de erro... -->

## Por quê
<!-- Link para o item do HANDOFF.md, a spec ou a issue que originou -->

## Tipo de mudança
- [ ] Aditiva (campo/endpoint novo e opcional), sem quebrar nada
- [ ] Quebra compatibilidade (renomear, remover, tornar obrigatório), exige coordenar backend e frontend

## Impacto
- Backend: <!-- o que precisa mudar -->
- Frontend: <!-- o que precisa mudar -->
- Mocks MSW: <!-- atualizar handlers? -->

## Checklist do arquiteto
- [ ] `docs/api/openapi.yaml` atualizado e válido
- [ ] `packages/shared` atualizado (schemas + tipos)
- [ ] `info.version` do contrato incrementada
- [ ] Spec e DOMAIN.md atualizados, se for o caso
- [ ] Issues de follow-up criadas para backend e frontend
