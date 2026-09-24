# ARCHITECTURE.md

> **Status: RASCUNHO ESTRUTURAL.** As decisões marcadas como *pendente* dependem de respostas
> de negócio registradas em `docs/HANDOFF.md` e serão fechadas via ADR.
> Nada aqui deve ser tratado como decidido até existir ADR correspondente.

## 1. Contexto

`agendafacil` parte de um repositório vazio (commit base `02fad20`, apenas `README.md`).
Nenhuma restrição técnica pré-existente foi encontrada — o que dá liberdade, mas também exige
que as primeiras decisões fiquem registradas.

## 2. Forma pretendida (a confirmar)

Monorepo com separação entre entregáveis e bibliotecas:

```
apps/         → serviços/aplicações executáveis (ex.: api, web)
packages/     → código compartilhado (ex.: shared: tipos, enums, validação)
docs/         → specs, ADRs, contrato de API, progresso
```

Princípio: **contrato antes de código**. `docs/api/openapi.yaml` define a superfície HTTP;
`packages/shared` publica os tipos derivados; `apps/*` consomem ambos.

## 3. Decisões em aberto (cada uma vira um ADR)

| # | Decisão | Por que importa | Estado |
|---|---|---|---|
| 1 | Domínio do produto (agendamento de horários vs. agenda de contatos vs. outro) | Define todo o modelo de dados | **Aberta** — pergunta enviada ao humano |
| 2 | Single-tenant vs. multi-tenant (SaaS com várias organizações) | Define isolamento de dados, auth e schema | **Aberta** |
| 3 | Quem opera o agendamento (autoatendimento público vs. painel interno vs. ambos) | Define superfície pública, auth anônima e rate limiting | **Aberta** |
| 4 | Regra de disponibilidade e tratamento de conflito de horário | É a regra de negócio central do domínio | **Aberta** |
| 5 | Stack de backend e ferramenta de monorepo | Define `packages/shared` (TS) vs. módulo Python, CI, comandos | **Aberta** |
| 6 | Banco de dados e estratégia de migration | Bloqueia `area:dados` | **Aberta** (depende de 5) |
| 7 | Notificações (e-mail/WhatsApp/push) e fuso horário | Agendamento sem lembrete perde valor; fuso é fonte clássica de bug | **Aberta** |

## 4. Restrições assumidas (revisar se estiverem erradas)

- Fuso horário **nunca** é implícito: todo instante é armazenado em UTC e exibido no fuso da
  organização/usuário. (Vira ADR quando o domínio for confirmado.)
- IDs opacos e não sequenciais em endpoints públicos.
- Erros de API seguem um envelope único (a definir no `openapi.yaml`).

## 5. Riscos já visíveis

1. **Dupla reserva (double booking)** — exige decisão explícita sobre lock/transação e sobre o
   que acontece quando dois clientes clicam no mesmo horário.
2. **Fuso horário e horário de verão** — grade semanal em hora local vs. instantes UTC.
3. **Cancelamento de última hora** — política precisa estar na spec antes do código.
4. **Escopo rastejante** — mitigado pela seção "Fora do escopo" obrigatória em toda issue.
