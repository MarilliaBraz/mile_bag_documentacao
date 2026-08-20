# Implementation Plan: HU16 — Gerenciar planos de assinatura

**Branch**: `HU16-gerenciar-planos-assinatura` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU16-gerenciar-planos-assinatura/spec.md`

## Summary

CRUD de planos de assinatura (limite de motoristas ativos, limite de volume mensal de BDOs) e a operação de associação/troca de plano de um tenant. É uma entidade de configuração da plataforma (não por tenant) consumida por HU14 no cadastro e reconsultada onde quer que os limites de plano precisem ser verificados (FR-017 do spec-mãe).

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context). Específico desta HU:

**Storage**: `PlanoAssinatura` é uma tabela global da plataforma, sem discriminador de tenant e sem Row Level Security por tenant (é a própria plataforma, não dado de um tenant) — mas ainda assim restrita por perfil (só administrador da plataforma acessa o CRUD de planos).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates avaliados em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#constitution-check). Reavaliação específica desta HU:

| Gate | Avaliação |
|---|---|
| Os 3 pilares: parametrização, ambiente, versão | PASS — é a própria essência desta HU: limites de plano são dado configurável, nunca hardcoded no código (RN10) |
| Confiabilidade e rastreabilidade dos dados | PASS — histórico de troca de plano de um tenant deve ser preservado (auditoria de quando/por qual plano um tenant passou), coerente com o princípio de rastreabilidade |
| Simplicidade antes de generalidade | PASS — sem faturamento/cobrança nesta HU (ver Assumptions do spec); escopo restrito aos limites operacionais já exigidos pelo spec-mãe |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU16-gerenciar-planos-assinatura/
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
    ├── controller/       # endpoints de CRUD de planos e de troca de plano do tenant
    ├── service/          # troca de plano, validação de remoção com tenants associados
    ├── repository/
    └── model/            # entidade PlanoAssinatura, associação Tenant → PlanoAssinatura

mile_bag_app/src/features/back-office/
└── planos/               # tela de administração de planos (visão do provedor)
```

**Structure Decision**: Mantido em `business/tenant`, junto de HU14/HU15 — plano de assinatura é parte do mesmo domínio de onboarding/gestão de tenants, evitando um pacote `business/plano` isolado para uma entidade tão pequena.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
