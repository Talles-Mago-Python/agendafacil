# Spec: Horário de funcionamento e bloqueios

**Status:** aprovada · **Regras:** HB-01 a HB-04 · **ADR:** 004
**Contrato:** tags `business-hours`, `blocked-slots`

## Objetivo
O admin define quando o negócio atende (por dia da semana) e bloqueia intervalos específicos (almoço, folga, feriado). Essas duas informações alimentam o cálculo de disponibilidade.

## Endpoints

| Método | Rota | Auth | Resumo |
|---|---|---|---|
| GET | `/api/business-hours` | público | Retorna os 7 dias (ordenados 0–6) |
| PUT | `/api/business-hours` | ADMIN | Substitui os 7 dias de uma vez |
| GET | `/api/blocked-slots?from=&to=` | ADMIN | Lista os bloqueios que sobrepõem o intervalo |
| POST | `/api/blocked-slots` | ADMIN | Cria bloqueio |
| DELETE | `/api/blocked-slots/{id}` | ADMIN | Remove bloqueio, `204` |

## Horário de funcionamento

### Regras do PUT
- O body deve ter **exatamente 7 itens**, com `weekday` de 0 a 6, sem repetição. Caso contrário, retorna 422.
- Se `isOpen = true`, `opensAt` e `closesAt` são obrigatórios, no formato `HH:mm`, em múltiplos de 15 min e com `closesAt > opensAt`.
- Se `isOpen = false`, `opensAt` e `closesAt` são ignorados e salvos como `null`.
- Não cancela nem valida agendamentos existentes (HB-04).

### Tela: `/admin/configuracoes/horarios`
- 7 linhas (Domingo … Sábado), cada uma com um toggle "Aberto" e dois selects (abre/fecha, de 15 em 15 min).
- Um botão "Salvar" envia os 7 dias. Toast "Horários salvos".
- A seção pública "Horário de funcionamento" (rodapé da home) usa o `GET`.

## Bloqueios

### Regras do POST
- `startsAt < endsAt`, e as duas datas em ISO UTC.
- O bloqueio não pode terminar no passado (`endsAt > agora`). Caso contrário, retorna 422 `VALIDATION_ERROR`.
- Se sobrepuser agendamentos `CONFIRMED`, retorna **409 `CONFLICTS_WITH_APPOINTMENTS`** com `details.appointmentIds: string[]` (HB-02).
- A duração máxima de um bloqueio é de 31 dias.

### GET com filtro
- `from` e `to` são obrigatórios (ISO UTC), e o intervalo tem no máximo 62 dias.
- Retorna os bloqueios com `startsAt < to AND endsAt > from`, ordenados por `startsAt`.

### Tela: `/admin/bloqueios`
- Lista dos próximos bloqueios (padrão: hoje + 30 dias), com data, intervalo, motivo e o botão remover.
- Formulário com data de início, hora de início, data de fim, hora de fim, motivo e os atalhos:
  - **"Dia inteiro"**: preenche 00:00 até 00:00 do dia seguinte
  - **"Almoço hoje"**: preenche 12:00–13:00
- Se receber 409, mostra: *"Existem N agendamentos nesse período"* com um link para cada um na agenda.
- Os horários são digitados e exibidos no fuso do negócio e convertidos para UTC antes de enviar (ADR-004).

## Critérios de aceite
- [ ] O PUT com 6 dias ou com um `weekday` repetido retorna 422
- [ ] `closesAt <= opensAt` retorna 422 com `details.fields`
- [ ] `opensAt = "09:10"` retorna 422 (não é múltiplo de 15)
- [ ] Um dia fechado fica sem nenhum slot em `/availability` (teste de integração)
- [ ] Um bloqueio sobre um agendamento confirmado retorna 409 com os IDs corretos
- [ ] Um bloqueio sobre um agendamento **cancelado** é permitido
- [ ] Um bloqueio de 12:00–13:00 remove os slots que o sobrepõem, e um slot que termina às 12:00 continua disponível (DS-03)
- [ ] Cliente acessando `/blocked-slots` recebe 403
