# ADR 001: Monorepo com pnpm workspaces

- **Status:** aceito
- **Data:** 2026-09-24

## Contexto
O projeto será desenvolvido por vários agentes de IA em paralelo (frontend, backend, dados, devops). Precisamos que front e back compartilhem tipos e validações sem copiar código, e que cada agente tenha um território claro.

## Decisão
Um único repositório com **pnpm workspaces**:
`apps/web`, `apps/api` e `packages/shared`, com `prisma/` na raiz.

## Consequências
- ✅ Tipos e schemas Zod em `packages/shared`, importados por front e back (`@agendafacil/shared`).
- ✅ Cada pasta tem um agente dono, o que reduz conflitos de merge.
- ✅ Um só CI, um só lugar para issues e PRs.
- ⚠️ `package.json` da raiz e o `pnpm-lock.yaml` são pontos de conflito. **Só o devops altera a raiz.** Os outros agentes adicionam dependências apenas no `package.json` da própria app e resolvem conflito de lockfile com `pnpm install` após o rebase.
- ⚠️ Não vamos usar Turborepo/Nx no MVP. Os scripts usam `pnpm -r` / `pnpm --filter`.

## Alternativas consideradas
| Alternativa | Por que foi descartada |
|---|---|
| Dois repositórios (web e api) | Tipos duplicados, contrato divergente, dois CIs |
| Next.js full-stack (API Routes) | Mistura as áreas de front e back e dificulta a separação por agente |
| Turborepo | Complexidade extra desnecessária no tamanho atual |
