# Spec: Disponibilidade e agendamento

**Status:** aprovada · **Regras:** DS-01 a DS-04, AG-01 a AG-10 · **ADRs:** 004, 005
**Contrato:** tags `availability`, `appointments`

> Este é o **coração do sistema**. Qualquer dúvida de regra deve ser resolvida no `DOMAIN.md`.

## Histórias
- Como **cliente**, quero escolher um serviço, ver os dias e horários livres e confirmar em poucos toques.
- Como **cliente**, quero ver meus próximos agendamentos e o histórico, e cancelar se precisar.
- Como **admin**, quero ver a agenda do dia e da semana, cancelar e marcar os atendimentos como concluídos ou como falta.

## Endpoints

| Método | Rota | Auth | Resumo |
|---|---|---|---|
| GET | `/api/availability?serviceId=&date=YYYY-MM-DD` | público | Slots livres do serviço na data (fuso do negócio) |
| GET | `/api/appointments` | logado | Cliente: os próprios. Admin: todos, com filtros |
| POST | `/api/appointments` | logado | Cria agendamento |
| GET | `/api/appointments/{id}` | logado | Detalhe (o cliente vê só o próprio, e os outros retornam 404) |
| POST | `/api/appointments/{id}/cancel` | logado | Cancela |
| PATCH | `/api/appointments/{id}/status` | ADMIN | Marca `COMPLETED` ou `NO_SHOW` |

---

## 1. Disponibilidade

### Algoritmo (função pura `calculateAvailableSlots`)
Entrada: `date` (local), `durationMinutes`, `businessHours` do weekday, `appointments` CONFIRMED do dia, `blockedSlots` do dia, `now`, `slotInterval`, `timezone`.

```
1. se date < hoje(local) ou date > hoje + 30 dias  → []
2. se !businessHours.isOpen                        → []
3. abertura = date + opensAt (local → UTC); fechamento = date + closesAt (local → UTC)
4. para inicio = abertura; inicio + duracao <= fechamento; inicio += slotInterval:
       fim = inicio + duracao
       se inicio < now + 60min                         → pular
       se sobrepõe algum appointment [s,e): inicio < e && fim > s → pular
       se sobrepõe algum blockedSlot                   → pular
       adicionar {startsAt: inicio, endsAt: fim}
5. retornar lista ordenada
```

- Para que a função seja testável, `now` **deve** ser um parâmetro.
- Resposta: `{ serviceId, date, timezone: "America/Sao_Paulo", slots: [{ startsAt, endsAt }] }`.
- Erros: um `serviceId` inexistente ou inativo retorna `404 NOT_FOUND`. Um `date` inválido retorna `422 VALIDATION_ERROR`.
- Uma data fora do intervalo permitido retorna `200` com `slots: []` (não é erro).

### Casos de teste obrigatórios (unitários)
| # | Cenário | Esperado |
|---|---|---|
| 1 | Dia fechado | `[]` |
| 2 | 09:00–12:00, serviço de 60 min, sem ocupação, intervalo de 15 | 09:00, 09:15 … 11:00 (9 slots) |
| 3 | Agendamento 10:00–10:30 e serviço de 30 min | 09:30 disponível, 09:45 não, 10:00 não, 10:15 não, 10:30 disponível |
| 4 | Bloqueio 12:00–13:00 e serviço de 60 min | 11:00 disponível, 11:15 não, … 13:00 disponível |
| 5 | `now` = 09:20 do mesmo dia | o primeiro slot é 10:30 (≥ 10:20, alinhado a 15) |
| 6 | Data 31 dias à frente | `[]` |
| 7 | Um agendamento CANCELLED no horário | não bloqueia |
| 8 | Serviço de 480 min num dia de 4h | `[]` |

## 2. Criar agendamento

`POST /api/appointments` com o body `{ serviceId, startsAt, notes? }`

Ordem das validações no `service.ts`:
1. Schema (Zod) → `422 VALIDATION_ERROR`
2. O serviço existe → `404`. Está ativo → `422 SERVICE_INACTIVE` (AG-05)
3. `startsAt >= now + 60min` → `422 BOOKING_TOO_SOON`
4. `startsAt <= hoje + 30 dias` → `422 BOOKING_TOO_FAR`
5. O cliente tem menos de 3 CONFIRMED futuros → `422 MAX_APPOINTMENTS_REACHED` (AG-04)
6. `startsAt` está entre os slots de `calculateAvailableSlots` → `409 SLOT_UNAVAILABLE`
7. INSERT numa **transação**. Se o banco violar a exclusion constraint (ADR-005), retorna `409 SLOT_UNAVAILABLE`

O agendamento é criado com `endsAt = startsAt + duração`, `status = CONFIRMED` e cópias de `priceCents` e `serviceName`. Resposta `201` com o `Appointment` (incluindo `service` e `client` resumidos).

## 3. Listar agendamentos

`GET /api/appointments?from=&to=&status=&page=&pageSize=`

- **Cliente:** sempre filtrado por `clientId = eu`. Os filtros são opcionais.
- **Admin:** vê todos. Pode filtrar por `from`/`to` (sobreposição com `startsAt`), `status` (repetível: `status=CONFIRMED&status=COMPLETED`) e `clientId`.
- Ordenação: `startsAt` ascendente. Padrão `page=1`, `pageSize=20`, máximo 100.

## 4. Cancelar

`POST /api/appointments/{id}/cancel` (sem body)

| Quem | Condição | Resultado |
|---|---|---|
| Cliente dono | CONFIRMED e `startsAt - now >= 2h` | 200, `status=CANCELLED`, `cancelledBy=CLIENT` |
| Cliente dono | CONFIRMED e faltam menos de 2h | 422 `CANCELLATION_WINDOW_EXPIRED` |
| Cliente não dono | – | 404 |
| Admin | CONFIRMED e `startsAt > now` | 200, `cancelledBy=ADMIN` |
| Qualquer um | status ≠ CONFIRMED | 409 `INVALID_STATUS_TRANSITION` |

## 5. Atualizar status (admin)

`PATCH /api/appointments/{id}/status` com `{ status: "COMPLETED" | "NO_SHOW" }`

- É preciso `status == CONFIRMED` e `startsAt <= now`. Caso contrário, retorna `409 INVALID_STATUS_TRANSITION`.
- Qualquer outro valor de `status` no body retorna 422.

---

## Telas

### Cliente: fluxo de agendamento (4 passos)
1. **`/servicos`**: o cliente escolhe o serviço e toca em "Agendar".
2. **`/agendar/{serviceId}`**:
   - resumo do serviço no topo (nome, duração, preço);
   - carrossel horizontal com os próximos 14 dias ("Seg 06/10"); os dias fechados aparecem desabilitados;
   - ao tocar num dia, chama `GET /availability` e mostra uma grade de horários ("09:00", "09:15"…);
   - estado vazio: "Sem horários neste dia. Tente outra data.";
   - se o usuário não estiver logado, ao tocar num horário ele vai para `/login?next=...`, preservando a escolha.
3. **Confirmação** (modal ou etapa): serviço, data por extenso ("segunda-feira, 6 de outubro, 09:30"), preço, campo "Observações (opcional)" e o botão **"Confirmar agendamento"**.
4. **Sucesso**: "Agendamento confirmado! ✅", resumo e os botões "Ver meus agendamentos" e "Agendar outro".

Tratamento de erros na confirmação:
- `409 SLOT_UNAVAILABLE`: *"Alguém acabou de reservar este horário. Escolha outro."* A tela recarrega os horários e volta ao passo 2.
- `422 MAX_APPOINTMENTS_REACHED`: *"Você já tem 3 agendamentos futuros. Cancele um para marcar outro."*
- `422 BOOKING_TOO_SOON`: *"Agendamentos precisam de pelo menos 1 hora de antecedência."*

### Cliente: `/meus-agendamentos`
- Abas **Próximos** (CONFIRMED futuros) e **Histórico** (o restante, do mais recente para o mais antigo).
- Card com serviço, data e hora, preço, status (badge) e o botão "Cancelar" (visível só se faltarem pelo menos 2h, mas o backend revalida).
- Cancelar pede confirmação e depois atualiza a lista.

### Admin: `/admin` (agenda)
- A abertura padrão é a **agenda de hoje**. Alternância **Dia / Semana**, com setas ← → e o botão "Hoje".
- **Dia:** linha do tempo das 06:00 às 22:00 com blocos de agendamentos (cliente, serviço, horário, cor pelo status) e bloqueios hachurados.
- **Semana:** 7 colunas com os mesmos blocos, em versão compacta. No celular, mostra uma lista agrupada por dia.
- Ao clicar num agendamento, abre um painel lateral com os dados do cliente (nome, telefone com link `tel:`), observações e as ações:
  - "Cancelar" (se futuro)
  - "Concluído" / "Faltou" (se já começou)
- Os agendamentos fora do horário de funcionamento atual (HB-04) aparecem com um ícone de alerta.
- Resumo do dia no topo: total de agendamentos e faturamento previsto (soma de `priceCents` dos CONFIRMED + COMPLETED).

## Critérios de aceite (integração)
- [ ] Os 8 casos da tabela de disponibilidade passam como testes unitários
- [ ] Dois `POST /appointments` **simultâneos** para o mesmo horário resultam em exatamente um 201 e um 409 (teste com `Promise.all`)
- [ ] O cliente A não vê nem cancela um agendamento do cliente B (404)
- [ ] O 4º agendamento futuro do mesmo cliente retorna 422 `MAX_APPOINTMENTS_REACHED`
- [ ] O cancelamento 1h59 antes pelo cliente retorna 422. Pelo admin, retorna 200
- [ ] Marcar `COMPLETED` num agendamento futuro retorna 409
- [ ] Um horário cancelado volta a aparecer em `/availability`
- [ ] `priceCents` do agendamento não muda depois de editar o preço do serviço
- [ ] Todas as datas da resposta estão em ISO UTC, terminando com `Z`
