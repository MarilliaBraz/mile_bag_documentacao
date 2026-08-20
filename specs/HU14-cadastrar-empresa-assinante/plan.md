# Implementation Plan: HU14 — Cadastrar empresa assinante (tenant)

**Branch**: `HU14-cadastrar-empresa-assinante` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU14-cadastrar-empresa-assinante/spec.md`

## Summary

Endpoint e tela administrativa (restritos ao administrador da plataforma) para criar um tenant associado a um plano de assinatura, com isolamento de dados garantido desde o primeiro registro e sem qualquer etapa de implantação dedicada. É a operação fundacional de onboarding, consumida em seguida por HU15 (convite ao primeiro administrador).

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context). Específico desta HU:

**Storage**: criação do tenant e da política de Row Level Security correspondente ocorrem na mesma transação (FR-006 — cadastro atômico); a política de RLS já deve existir de antemão para a tabela de tenants e ser parametrizada pelo novo `tenantId`, não criada por tenant (RLS é definida uma vez no schema, filtrando por coluna discriminadora — ver plano-mãe, seção 3, "Isolamento multi-tenant").

**Constraints**: nenhum passo de deploy, criação de schema ou provisionamento de infraestrutura pode fazer parte do fluxo desta HU (FR-005) — é puramente um `INSERT` de aplicação sobre a base compartilhada já existente.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates avaliados em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#constitution-check). Reavaliação específica desta HU:

| Gate | Avaliação |
|---|---|
| Segurança e privacidade por padrão / isolamento entre tenants | PASS — é justamente esta HU que estabelece o registro isolado de cada tenant (FR-004); testes de isolamento (SC-002) fazem parte da definição de pronto |
| Simplicidade antes de generalidade | PASS — cadastro é uma operação simples (criar tenant + associar plano); nenhuma etapa de provisionamento de infraestrutura é introduzida (proibido por FR-005) |
| Os 3 pilares: parametrização, ambiente, versão | PASS — plano de assinatura é referenciado por associação (não copiado/hardcoded); tenant é apenas dado de aplicação, sem variação por ambiente |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU14-cadastrar-empresa-assinante/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/
└── tenant/
    ├── controller/      # endpoint de cadastro (uso restrito a administrador da plataforma)
    ├── service/         # criação atômica do tenant + associação ao plano
    ├── repository/
    └── model/           # entidade Tenant

mile_bag_app/src/features/back-office/
└── tenant-onboarding/   # tela de cadastro de empresa assinante (visão do provedor)
```

**Structure Decision**: Reaproveita o subpacote `business/tenant` já previsto no plano-mãe. Não há necessidade de pacote próprio — HU14, HU15 e HU16 compartilham o mesmo domínio `tenant`.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
