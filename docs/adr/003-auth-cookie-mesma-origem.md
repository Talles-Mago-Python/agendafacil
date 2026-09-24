# ADR 003: Sessão por JWT em cookie httpOnly, com mesma origem via rewrite do Next

- **Status:** aceito
- **Data:** 2026-09-24

## Contexto
Web (Vercel) e API (Render) ficam em domínios diferentes. Guardar o token em `localStorage` expõe o token a XSS. Cookies entre domínios diferentes exigem `SameSite=None` e CORS com credenciais, e os navegadores estão bloqueando cookies de terceiros.

## Decisão
1. O Next.js faz **rewrite** de `/api/:path*` para `${API_URL}/api/:path*`. O navegador só enxerga o domínio do site.
2. A API emite o cookie **`af_session`** com um JWT (HS256, `jose`) contendo `{ sub, role }`:
   `HttpOnly; SameSite=Lax; Path=/; Max-Age=<SESSION_TTL_DAYS>; Secure` em produção.
3. O frontend usa **somente URLs relativas** (`fetch('/api/...')`) e nunca lê o token.
4. Senhas com **argon2id**.

## Consequências
- ✅ Sem CORS, sem token acessível a JavaScript e cookie first-party.
- ✅ A proteção contra CSRF vem do `SameSite=Lax` + requisições JSON (`Content-Type: application/json`). A API rejeita mutações com outro content-type.
- ⚠️ Logout "global" e revogação de sessão não existem no MVP (o JWT vale até expirar). Aceitável para a v1.
- ⚠️ Em desenvolvimento, o Next (3000) também faz o rewrite para a API (3333). **Nunca** use `http://localhost:3333` no código do navegador.
- ⚠️ A API deve confiar no header `X-Forwarded-For` (proxy do Next e da Vercel) para o rate limit funcionar por IP (`trustProxy: true`).

## Alternativas consideradas
| Alternativa | Por que foi descartada |
|---|---|
| JWT em localStorage + header Authorization | Vulnerável a XSS |
| Cookie cross-site `SameSite=None` + CORS | Frágil com o bloqueio de cookies de terceiros |
| Sessões em banco (tabela sessions) | Mais seguro para revogação, mas adiado para a v2 |
| Auth.js / Clerk | Acoplaria a auth ao frontend ou a um serviço pago |
