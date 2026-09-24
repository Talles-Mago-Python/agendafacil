# Spec: Autenticação

**Status:** aprovada · **Regras:** AC-01 a AC-05 (`DOMAIN.md`) · **ADR:** 003
**Contrato:** `openapi.yaml` → tag `auth`

## Objetivo
Permitir que clientes criem conta, entrem e saiam, e que o sistema saiba quem é o usuário e qual o papel dele em cada requisição.

## Histórias
- Como **visitante**, quero criar uma conta com nome, e-mail, telefone e senha para poder agendar.
- Como **usuário**, quero entrar com e-mail e senha e continuar logado por alguns dias.
- Como **usuário**, quero sair da minha conta.
- Como **admin**, quero ser levado direto ao painel depois de entrar.

## Endpoints

| Método | Rota | Auth | Resumo |
|---|---|---|---|
| POST | `/api/auth/register` | público | Cria cliente e já inicia sessão |
| POST | `/api/auth/login` | público | Inicia sessão |
| POST | `/api/auth/logout` | público | Encerra sessão (idempotente) |
| GET | `/api/auth/me` | logado | Retorna o usuário atual |

## Fluxos

### Cadastro
1. O visitante preenche nome, e-mail, telefone (opcional), senha e confirmação de senha.
2. O frontend valida com o schema do shared (a confirmação de senha é validada só no front).
3. `POST /auth/register` → `201` + cookie `af_session` + `User`.
4. O frontend redireciona para `/agendar` (ou para a página que o usuário tentou acessar antes).

**Erros:** `409 EMAIL_ALREADY_EXISTS` (mensagem no campo e-mail), `422 VALIDATION_ERROR` e `429 RATE_LIMITED`.

### Login
1. `POST /auth/login` com e-mail e senha → `200` + cookie + `User`.
2. Se `role = ADMIN`, redireciona para `/admin`. Se não, para `/meus-agendamentos`.

**Erros:** `401 INVALID_CREDENTIALS` com a mensagem genérica (AC-05) e `429 RATE_LIMITED`.

### Sessão
- O cookie `af_session` guarda um JWT assinado (HS256) com `{ sub: userId, role }` e `exp = agora + SESSION_TTL_DAYS`.
- Atributos: `HttpOnly; SameSite=Lax; Path=/; Secure` (em produção); `Max-Age` igual ao TTL.
- Em cada request, o plugin de auth valida o JWT e carrega o usuário. Se o usuário não existir mais, retorna 401.
- No carregamento inicial, o frontend chama `GET /auth/me`: `200` significa logado e `401` significa visitante.

### Logout
`POST /auth/logout` apaga o cookie (`Max-Age=0`) e retorna `204`, mesmo sem sessão.

## Proteção de rotas no frontend

| Rota | Quem acessa | Se não puder |
|---|---|---|
| `/`, `/servicos`, `/login`, `/cadastro` | todos | – |
| `/agendar/**`, `/meus-agendamentos` | CLIENT ou ADMIN | redireciona para `/login?next=<rota>` |
| `/admin/**` | ADMIN | visitante vai para `/login`, e cliente vai para `/` |

> O backend **sempre** revalida. A proteção no front serve apenas para a experiência do usuário.

## Telas
- **/cadastro**: campos nome, e-mail, telefone (máscara `(12) 99999-9999`), senha com botão mostrar/ocultar e confirmar senha. Link "Já tenho conta".
- **/login**: e-mail, senha, botão "Entrar", link "Criar conta".
- **Cabeçalho**: visitante vê "Entrar". Cliente vê "Meus agendamentos" e "Sair". Admin vê "Painel" e "Sair".

## Critérios de aceite
- [ ] O cadastro cria `role = CLIENT`, mesmo que o body envie `role: "ADMIN"` (o campo é ignorado)
- [ ] O e-mail é salvo em minúsculas, e `Joao@X.com` e `joao@x.com` conflitam
- [ ] A senha fraca é rejeitada com `VALIDATION_ERROR` e `details.fields.password`
- [ ] `passwordHash` nunca aparece em nenhuma resposta
- [ ] Login errado retorna a mesma mensagem para e-mail inexistente e para senha errada
- [ ] A 6ª tentativa de login em 1 minuto no mesmo IP retorna 429
- [ ] O cookie tem `HttpOnly` e `SameSite=Lax`
- [ ] `GET /auth/me` sem cookie retorna 401 `UNAUTHENTICATED`
- [ ] O logout funciona mesmo sem sessão (204)

## Fora do escopo
Recuperação de senha, verificação de e-mail, login social, edição de perfil (v2).
