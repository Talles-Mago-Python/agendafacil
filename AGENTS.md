# AGENTS.md

Contrato de operação dos agentes deste repositório (`Talles-Mago-Python/agendafacil`).
Leia este arquivo antes de mudar código **ou** documentação.

> Status: esqueleto de processo criado pelo Agente Arquiteto. A visão de produto
> (`docs/VISION.md`) e as decisões de stack (ADRs) ainda **não** foram fechadas —
> veja "Decisões em aberto" em `docs/ARCHITECTURE.md`.

---

## 1. Papéis e fronteiras de escrita

| Agente | Responsabilidade | Pode escrever em | **Não** pode escrever em |
|---|---|---|---|
| **Arquiteto** | Ideia → spec → contrato → tipos → ADRs → issues | `docs/specs/`, `docs/api/openapi.yaml`, `packages/shared/`, `docs/adr/`, `docs/ARCHITECTURE.md`, `docs/PROGRESS.md`, `docs/HANDOFF.md`, `AGENTS.md`, issues no GitHub | `apps/**` (não implementa features) |
| **Dados** | Modelo físico, migrations, seeds, integridade | `apps/*/migrations`, `packages/shared` (apenas o que o Arquiteto delegar via issue) | specs, ADRs |
| **Backend** | API HTTP conforme `openapi.yaml` | `apps/api/**` | contrato da API (mudança de contrato = issue para o Arquiteto) |
| **Frontend** | UI/UX conforme spec | `apps/web/**` | contrato da API |
| **DevOps** | CI/CD, contêineres, ambientes, observabilidade | `.github/workflows/`, `docker*`, `infra/` | regras de negócio |
| **QA** | Plano de testes, e2e, regressão | `apps/*/tests`, `e2e/`, relatórios em `docs/` | código de produção |

Regra dura: **quem descobre uma ambiguidade de negócio não decide sozinho** — abre item em
`docs/HANDOFF.md` ou comenta na issue e espera o Arquiteto responder na spec.

## 2. Fluxo de uma feature

```
ideia (humano)
  → [Arquiteto] até 5 perguntas sobre regras de negócio ambíguas
  → [Arquiteto] docs/specs/NNNN-titulo.md            (o quê / por quê / regras)
  → [Arquiteto] docs/adr/NNNN-*.md                   (só se houver decisão estrutural)
  → [Arquiteto] docs/api/openapi.yaml                (contrato primeiro, código depois)
  → [Arquiteto] packages/shared                      (tipos/DTOs/enums derivados do contrato)
  → [Arquiteto] issues: 1 por área, labels area:*
  → [Dados/Backend/Frontend/DevOps] implementação
  → [QA] verificação contra critérios de aceite da spec
  → [Arquiteto] docs/PROGRESS.md atualizado + HANDOFF resolvido
```

Nada entra em `apps/` sem spec numerada e sem issue aberta com critérios de aceite.

## 3. Anatomia obrigatória de uma issue

Cada issue tem **exatamente** estas seções (template em `.github/ISSUE_TEMPLATE/feature.md`):

1. **Objetivo** — uma frase, verificável.
2. **Contexto** — link para `docs/specs/…`, trecho relevante da spec, estado atual.
3. **Critérios de aceite** — lista checável, testável por QA sem ler o código.
4. **Fora do escopo** — o que *não* será feito agora (evita escopo rastejante).
5. **Dependências** — issues/contratos que precisam estar prontos antes.

Uma issue = uma área = um entregável pequeno (ideal: < 1 dia de trabalho de agente).

### Labels de área

`area:dados` · `area:backend` · `area:frontend` · `area:devops` · `area:qa`

Complementares sugeridos: `spec:<NNNN>`, `blocked`, `ready`.
Estado em 24/09/2026: os labels `area:*` **ainda não existem** no repositório GitHub
(só os defaults do GitHub). Criá-los é tarefa do Arquiteto na primeira rodada de issues.

## 4. Definition of Done (por área)

- **Geral**: roda o comando de teste/lint do projeto localmente e sai verde; sem TODO órfão.
- **Backend**: endpoint implementado bate com `openapi.yaml` (path, método, schema, códigos de erro).
- **Frontend**: consome o tipo de `packages/shared`, não redefine DTO à mão.
- **Dados**: migration reversível + seed/fixture cobrindo o caso da spec.
- **QA**: cada critério de aceite da spec tem pelo menos um teste nomeado em sua homenagem.
- **Arquiteto**: spec sem seções `TBD`; contrato validado; issues ligadas à spec.

## 5. Convenções

- **Idioma da documentação**: português (BR). Código, identificadores e commits: inglês.
- **Commits**: Conventional Commits (`docs:`, `feat(spec):`, `chore:` …). Prefixo do agente opcional.
- **Branch desta sessão**: `arena/01a0d47e-agendafacil` (não criar/empurrar outras).
- **Numeramento**: specs e ADRs usam `NNNN` de 4 dígitos, sequencial e nunca reutilizado.
- **Fonte da verdade do contrato**: `docs/api/openapi.yaml`. Tipos em `packages/shared`
  são *derivados* dele — em conflito, vale o YAML.
- **Monorepo**: `apps/*` = entregáveis executáveis; `packages/*` = bibliotecas compartilhadas.
  (Ferramenta de workspace ainda é decisão em aberto — ver ADR pendente.)

## 6. Mapa do repositório

```
AGENTS.md                 este arquivo
README.md
docs/
  VISION.md               produto: para quem, dor, escopo do MVP        [PENDENTE]
  ARCHITECTURE.md         visão técnica + decisões em aberto
  PROGRESS.md             painel de status por feature/área
  HANDOFF.md              fila de pedidos ao Arquiteto (entrada/saída)
  specs/                  especificações numeradas (+ _TEMPLATE.md)
  adr/                    decisões de arquitetura (+ _TEMPLATE.md)
  api/openapi.yaml        contrato da API (stub v0.0.0)
packages/shared/          tipos compartilhados                          [PENDENTE]
apps/                     implementações                                [PENDENTE]
```

## 7. Comandos

Ainda não há ferramenta de build/teste no repositório (nenhum `package.json`, `pyproject.toml`
ou similar no commit base). Esta seção será preenchida assim que a stack for definida em ADR.
