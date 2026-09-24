# Handoff: pedidos entre agentes

> Use quando precisar de algo **fora da sua área**. Registre o pedido, avise o humano e **pare** (não implemente na área do outro).
> O arquiteto (ou o agente dono) responde e move o item para "Resolvidos".

## Como registrar

```markdown
### H-NNN · [de: área] → [para: área] · Título curto
- **Data:** AAAA-MM-DD
- **Issue relacionada:** #N
- **Preciso de:** o que exatamente (campo, endpoint, variável de ambiente, dependência...)
- **Motivo:** por que é necessário (link para a spec, se houver)
- **Bloqueia minha tarefa?** sim / não (sigo com um mock ou contorno)
- **Status:** aberto
```

## Exemplos de pedidos válidos
- O frontend precisa de um campo novo na resposta → **para: arquiteto** (muda o contrato)
- O backend precisa de um índice novo → **para: dados**
- Qualquer agente precisa de uma variável de ambiente nova ou de uma dependência na raiz → **para: devops**
- Uma regra de negócio ambígua ou contraditória → **para: arquiteto** (esclarece no DOMAIN.md)

---

## Abertos
_(nenhum)_

## Resolvidos
_(nenhum)_
