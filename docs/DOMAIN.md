# Domínio: entidades e regras de negócio

> **Fonte única das regras.** As specs referenciam as regras por código (ex.: `AG-03`).
> Mudou uma regra? Altere aqui primeiro, num PR do arquiteto.

## 1. Glossário

| Termo (PT) | Nome no código | Significado |
|---|---|---|
| Usuário | `User` | Pessoa com conta (cliente ou admin) |
| Serviço | `Service` | O que é oferecido (ex.: "Corte masculino", 30 min, R$ 45) |
| Agendamento | `Appointment` | Reserva de um serviço por um cliente num horário |
| Horário de funcionamento | `BusinessHours` | Janela de atendimento de cada dia da semana |
| Bloqueio | `BlockedSlot` | Intervalo em que não se atende (almoço, folga, feriado) |
| Horário disponível | `Slot` | Início possível para um serviço numa data (calculado, não salvo) |

## 2. Entidades

### User
| Campo | Tipo | Regras |
|---|---|---|
| id | uuid | PK |
| name | string | 2–100 caracteres |
| email | string | único, minúsculo, formato válido |
| phone | string? | opcional, só dígitos, 10–11 (DDD + número) |
| passwordHash | string | argon2id, nunca exposto na API |
| role | enum `CLIENT` \| `ADMIN` | padrão `CLIENT` |
| createdAt / updatedAt | datetime | automáticos |

### Service
| Campo | Tipo | Regras |
|---|---|---|
| id | uuid | PK |
| name | string | 2–80 caracteres, único entre serviços ativos |
| description | string? | até 500 caracteres |
| durationMinutes | int | 15–480, **múltiplo de 15** |
| priceCents | int | ≥ 0 (em centavos: R$ 45,00 = 4500) |
| active | boolean | padrão `true` |
| createdAt / updatedAt | datetime | automáticos |

### BusinessHours
| Campo | Tipo | Regras |
|---|---|---|
| weekday | int | 0 = domingo … 6 = sábado, PK (exatamente 7 registros) |
| isOpen | boolean | se `false`, o negócio não abre nesse dia |
| opensAt | string `HH:mm` | obrigatório se `isOpen`, múltiplo de 15 min |
| closesAt | string `HH:mm` | obrigatório se `isOpen`, > `opensAt` |

> Os horários são **horário local do negócio** (`BUSINESS_TIMEZONE`). Na v1 não há dois turnos no mesmo dia: o almoço é feito com um bloqueio recorrente manual.

### BlockedSlot
| Campo | Tipo | Regras |
|---|---|---|
| id | uuid | PK |
| startsAt | datetime (UTC) | – |
| endsAt | datetime (UTC) | > `startsAt` |
| reason | string? | até 200 caracteres (ex.: "Almoço", "Feriado") |
| createdById | uuid | FK → User (admin) |
| createdAt | datetime | automático |

### Appointment
| Campo | Tipo | Regras |
|---|---|---|
| id | uuid | PK |
| clientId | uuid | FK → User |
| serviceId | uuid | FK → Service |
| startsAt | datetime (UTC) | – |
| endsAt | datetime (UTC) | = `startsAt + service.durationMinutes` (copiado na criação) |
| status | enum | `CONFIRMED` \| `CANCELLED` \| `COMPLETED` \| `NO_SHOW` |
| notes | string? | até 500 caracteres, escrito pelo cliente |
| priceCents | int | **cópia** do preço do serviço na criação |
| serviceName | string | **cópia** do nome do serviço na criação |
| cancelledAt | datetime? | preenchido ao cancelar |
| cancelledBy | enum? | `CLIENT` \| `ADMIN` |
| createdAt / updatedAt | datetime | automáticos |

> Preço e nome são copiados para o histórico não mudar quando o serviço for editado.

### Diagrama

```
User 1 ───< Appointment >─── 1 Service
User(admin) 1 ───< BlockedSlot
BusinessHours (7 linhas, sem relação)
```

## 3. Regras de negócio

### Contas e acesso (AC)
- **AC-01** O cadastro público sempre cria `role = CLIENT`. Admins são criados pelo seed ou pelo banco.
- **AC-02** O e-mail é único (comparação sem diferenciar maiúsculas).
- **AC-03** A senha tem no mínimo 8 caracteres, com pelo menos 1 letra e 1 número.
- **AC-04** O cliente só vê e altera os **próprios** agendamentos. O admin vê todos.
- **AC-05** Uma falha de login retorna uma mensagem genérica ("E-mail ou senha inválidos"), sem revelar qual dos dois está errado.

### Serviços (SV)
- **SV-01** Visitantes e clientes veem somente os serviços `active = true`.
- **SV-02** Serviços **não são apagados**. "Excluir" = desativar (`active = false`).
- **SV-03** Desativar um serviço **não cancela** os agendamentos futuros. O admin decide caso a caso.
- **SV-04** Editar a duração ou o preço não altera agendamentos já existentes.

### Horários e bloqueios (HB)
- **HB-01** Sempre existem 7 registros de `BusinessHours`. O padrão do seed é seg–sex 09:00–18:00, sáb 09:00–13:00 e dom fechado.
- **HB-02** Um bloqueio não pode ser criado se sobrepuser um agendamento `CONFIRMED`. A API retorna os conflitos e o admin cancela antes, se quiser.
- **HB-03** Bloqueios podem se sobrepor entre si (não há problema).
- **HB-04** Alterar o horário de funcionamento **não** cancela os agendamentos existentes fora da nova janela. Eles continuam válidos e aparecem destacados na agenda do admin.

### Disponibilidade (DS)
- **DS-01** Os horários candidatos começam em `opensAt` e avançam de `SLOT_INTERVAL_MINUTES` em `SLOT_INTERVAL_MINUTES` (padrão 15).
- **DS-02** Um slot `[início, início + duração)` é disponível se **todas** as condições valem:
  1. está inteiro dentro de `[opensAt, closesAt)` do dia;
  2. não sobrepõe nenhum `Appointment` com status `CONFIRMED`;
  3. não sobrepõe nenhum `BlockedSlot`;
  4. `início ≥ agora + 1 hora` (antecedência mínima);
  5. a data não passa de **30 dias** a partir de hoje.
- **DS-03** Intervalos são semiabertos: um agendamento que termina às 10:00 **não** conflita com outro que começa às 10:00.
- **DS-04** Status `CANCELLED` liberam o horário. `COMPLETED` e `NO_SHOW` estão no passado e são irrelevantes.

### Agendamentos (AG)
- **AG-01** Só usuários autenticados agendam. O cliente agenda para si mesmo.
- **AG-02** Para criar um agendamento, o slot precisa ser disponível pela regra DS-02 **no momento da criação**. A checagem acontece de novo no servidor, e o valor mostrado na tela não conta.
- **AG-03** **Nunca** existem dois `CONFIRMED` sobrepostos. Isso é garantido **também no banco** (exclusion constraint, ADR-005). Em caso de corrida, o segundo recebe `409 SLOT_UNAVAILABLE`.
- **AG-04** Um cliente pode ter no máximo **3** agendamentos `CONFIRMED` futuros.
- **AG-05** O serviço precisa estar `active` no momento da criação.
- **AG-06** O cliente pode cancelar até **2 horas antes** do início. Depois disso retorna `422 CANCELLATION_WINDOW_EXPIRED`.
- **AG-07** O admin pode cancelar a qualquer momento, antes do início.
- **AG-08** Só se cancela um agendamento `CONFIRMED`.
- **AG-09** O admin marca `COMPLETED` ou `NO_SHOW` somente **depois do início** do agendamento e somente se ele estiver `CONFIRMED`.
- **AG-10** As transições de status permitidas são:

```
CONFIRMED ──► CANCELLED   (cliente até 2h antes | admin antes do início)
CONFIRMED ──► COMPLETED   (admin, após o início)
CONFIRMED ──► NO_SHOW     (admin, após o início)
(todos os outros estados são finais)
```

## 4. Datas e fuso (resumo do ADR-004)
- O banco e a API usam **UTC** em ISO 8601 (`2026-10-05T12:00:00.000Z`).
- `BusinessHours` e o parâmetro `date` de `/availability` estão no **fuso do negócio**.
- O frontend exibe tudo no fuso do negócio (`America/Sao_Paulo`), e não no fuso do navegador.
