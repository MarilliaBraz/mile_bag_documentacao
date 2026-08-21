# mile_bag_documentacao

Repositório de documentação do projeto **Milebag — Gestão de Última Milha de Bagagens**, sistema web SaaS multi-tenant para digitalizar a devolução de bagagens extraviadas no transporte aéreo brasileiro (TCC do curso de Análise e Desenvolvimento de Sistemas, Faculdade SENAI FATESG).

## Estrutura

```
mile_bag_documentacao/
├── evidencias/    # Documentos formais do projeto (TCC, TAP, etc.)
├── requisitos/    # Especificação de requisitos, derivada das evidências
├── HUs/           # Histórias de usuário, derivadas dos requisitos funcionais
├── specs/         # Especificações spec-kit (spec.md por feature), derivadas das HUs
└── sessao/        # Registro do que foi feito em cada sessão de trabalho
```

### `evidencias/`

Documentos oficiais do trabalho acadêmico e do projeto, incluindo o documento principal (introdução, escopo, Termo de Abertura do Projeto — TAP), nos formatos original (`.docx`) e em Markdown para leitura/versionamento mais simples.

### `requisitos/`

Especificação de requisitos funcionais, não funcionais, regras de negócio, regras de domínio e regras de interface, extraída e detalhada a partir do documento em `evidencias/`.

### `HUs/`

Histórias de usuário, organizadas por persona (motorista, administrador do tenant, administrador da plataforma), derivadas dos requisitos funcionais em `requisitos/`.

### `specs/`

Especificações no formato spec-kit, geradas a partir das HUs em `HUs/`. Usado pelo fluxo `/speckit-*` (Claude Code) para planejamento e geração de tarefas.

- `001-milebag-user-stories/` — spec consolidada com as 16 HUs agrupadas em 3 User Stories priorizadas (visão geral do produto).
- `HU01-.../` a `HU16-.../` — uma pasta por HU, cada uma com o ciclo completo do spec-kit: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` e `tasks.md`. `plan.md`/`research.md` de cada HU referenciam por link relativo os equivalentes em `001-milebag-user-stories/` para não repetir a definição da stack e da Constitution Check. `tasks.md` segue a fase única User Story 1 (a própria HU) — Setup → Foundational → US1 🎯 MVP → Polish —, sem tarefas de teste (não solicitadas nos specs).

### `sessao/`

Um arquivo por sessão de trabalho (`AAAA-MM-DD.md`), documentando o que foi feito, decisões tomadas e próximos passos — histórico de andamento do projeto ao longo do tempo.

## Repositórios relacionados

- [`mile_bag_api`](https://github.com/MarilliaBraz/mile_bag_api) — back-end (Java/Spring Boot)
- [`mile_bag_app`](https://github.com/MarilliaBraz/mile_bag_app) — front-end / aplicação do motorista (React)
- [`mile_bag_prototipo`](https://github.com/MarilliaBraz/mile_bag_prototipo) — protótipo navegável (React) das 16 HUs, sem back-end real
- [`mile_bag_infra`](https://github.com/MarilliaBraz/mile_bag_infra) — infraestrutura e implantação
- [`mile_bag_audite`](https://github.com/MarilliaBraz/mile_bag_audite) — constituição e convenções técnicas do projeto
