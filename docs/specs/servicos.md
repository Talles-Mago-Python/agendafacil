# Spec: Serviços

**Status:** aprovada · **Regras:** SV-01 a SV-04 · **Contrato:** tag `services`

## Objetivo
O admin mantém o catálogo de serviços, e visitantes e clientes veem os serviços ativos com duração e preço.

## Histórias
- Como **visitante**, quero ver a lista de serviços com preço e duração antes de criar conta.
- Como **admin**, quero cadastrar, editar e desativar serviços.
- Como **admin**, quero reativar um serviço desativado.

## Endpoints

| Método | Rota | Auth | Resumo |
|---|---|---|---|
| GET | `/api/services` | público | Lista os serviços ativos. O admin pode passar `?includeInactive=true` |
| GET | `/api/services/{id}` | público | Detalhe (um inativo só aparece para o admin, e para os outros retorna 404) |
| POST | `/api/services` | ADMIN | Cria |
| PATCH | `/api/services/{id}` | ADMIN | Edita parcialmente (inclui `active` para reativar) |
| DELETE | `/api/services/{id}` | ADMIN | Desativa (`active=false`), `204` |

- A lista vem ordenada por `name` (A–Z) e não é paginada (espera-se menos de 50 serviços).
- `includeInactive=true` enviado por quem não é admin é **ignorado** (não gera erro).

## Validações (ver `DOMAIN.md` → Service)
- `name`: 2–80 caracteres, espaços nas pontas removidos, único entre os ativos (`409 SERVICE_NAME_EXISTS`)
- `durationMinutes`: 15–480, múltiplo de 15
- `priceCents`: inteiro ≥ 0
- `description`: até 500 caracteres

## Telas

### Público: `/servicos` (e a seção na home)
- Cards com nome, descrição (2 linhas, com reticências), duração ("30 min", "1h 30min"), preço ("R$ 45,00") e o botão **"Agendar"**, que leva a `/agendar/{serviceId}`.
- Estado vazio: "Nenhum serviço disponível no momento."

### Admin: `/admin/servicos`
- Tabela/lista com nome, duração, preço, status (badge Ativo/Inativo) e ações (Editar, Desativar/Reativar).
- Filtro "Mostrar inativos".
- Botão "Novo serviço" abre um formulário (modal ou página `/admin/servicos/novo`).
- Formulário: nome, descrição, duração (select de 15 em 15 até 8h) e preço (input em reais, convertido para centavos).
- Desativar pede confirmação: *"Clientes não poderão mais agendar este serviço. Agendamentos já marcados continuam válidos."*

## Critérios de aceite
- [ ] Visitante recebe só os serviços ativos, mesmo com `?includeInactive=true`
- [ ] O admin com `?includeInactive=true` recebe todos
- [ ] `DELETE` não apaga a linha: marca `active=false` e retorna 204
- [ ] `DELETE` de um serviço já inativo retorna 204 (idempotente)
- [ ] `durationMinutes = 20` retorna 422 `VALIDATION_ERROR`
- [ ] Criar "Corte" quando já existe "Corte" ativo retorna 409. Com "Corte" inativo, é permitido
- [ ] Cliente ou visitante em POST/PATCH/DELETE recebe 401 (sem sessão) ou 403 (cliente)
- [ ] O preço aparece formatado em BRL, e o admin digita em reais ("45,00" → 4500)
- [ ] Editar o preço não altera o `priceCents` de agendamentos existentes (SV-04)
