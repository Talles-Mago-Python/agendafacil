# ADR 002: Stack principal (Next.js, Fastify, Prisma, PostgreSQL)

- **Status:** aceito
- **Data:** 2026-09-24

## Contexto
Precisamos de uma stack popular (os modelos de IA a conhecem bem), tipada de ponta a ponta e com deploy barato.

## Decisão
- **Frontend:** Next.js (App Router) + Tailwind + TanStack Query
- **Backend:** Fastify + Zod
- **Banco:** PostgreSQL 16 com Prisma como ORM e ferramenta de migrações
- **Linguagem:** TypeScript strict em tudo

## Consequências
- ✅ Tipagem do banco até a tela (Prisma → shared → web).
- ✅ Os agentes de IA produzem código de boa qualidade nesse ecossistema.
- ✅ Postgres permite a exclusion constraint contra sobreposição (ADR-005).
- ⚠️ O Prisma não modela exclusion constraints, que ficam em SQL cru na migração (responsabilidade do agente dados).
- ⚠️ Next.js é usado **só como frontend**. Não criar API Routes com regra de negócio em `apps/web`.

## Alternativas consideradas
| Alternativa | Por que foi descartada |
|---|---|
| NestJS | Mais cerimônia e boilerplate. Fastify basta para o tamanho do MVP |
| Drizzle ORM | Boa opção, mas o Prisma tem migrações e seed mais simples para iniciantes |
| MongoDB | Agendamento é relacional e exige garantias de concorrência |
| Supabase (BaaS) | Regras de negócio complexas ficariam espalhadas em RLS e functions |
