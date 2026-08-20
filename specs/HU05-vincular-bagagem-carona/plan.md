# Implementation Plan: Vincular bagagem carona à entrega

**Branch**: `HU05-vincular-bagagem-carona` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU05-vincular-bagagem-carona/spec.md`

## Summary

Permite ao motorista vincular, durante uma entrega em andamento, bagagens adicionais (carona) sem abrir uma viagem completa separada, remunerando apenas a quilometragem complementar. Reaproveita a mesma entrega/viagem já em execução (HU01–HU04); não introduz stack nova além do já definido em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md).

## Technical Context

Herda integralmente o Technical Context do plano-mãe ([`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context)): back-end Java 21/Spring Boot 3 (`mile_bag_api`), front-end React/TypeScript PWA (`mile_bag_app`), PostgreSQL+PostGIS com Row Level Security. Nada específico desta HU exige uma dependência, storage ou plataforma adicional.

**Constraints específicas**: a operação de vincular bagagem carona precisa funcionar também offline (mesma fila local de HU08), já que ocorre durante a execução da viagem em campo.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Reavaliação pontual dos gates já descritos em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#constitution-check) (fonte: `mile_bag_audite/CONSTITUTION.md` e `CONVENTIONS.md`):

| Gate | Avaliação para esta HU |
|---|---|
| Simplicidade antes de generalidade | PASS — bagagem carona é modelada como entidade simples associada à entrega principal, sem generalizar para "múltiplas entregas por viagem" além do que RF05 exige |
| Confiabilidade e rastreabilidade dos dados | PASS — vínculo e quilometragem complementar ficam registrados, não recalculados a posteriori |
| Separação `core`/`business` (backend) | PASS — lógica de vínculo entra em `business/entrega`, reutilizando `business/bdo` para a captura do BDO da bagagem carona |
| Organização por `feature` (frontend) | PASS — UI de vínculo entra como parte da feature `motorista/captura-bdo` ou uma sub-tela de `motorista/rota` (entrega em andamento) |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU05-vincular-bagagem-carona/
├── spec.md
├── plan.md          # este arquivo
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes dos repositórios do projeto)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/
└── bagagemcarona/            # vínculo entrega principal ↔ bagagem carona, cálculo de km complementar

mile_bag_app/src/features/motorista/
└── captura-bdo/
    └── bagagem-carona/       # tela/ação de vincular bagagem carona a uma entrega em andamento
```

**Structure Decision**: Reaproveita os pacotes/domínios já previstos no plano-mãe (`business/entrega`, `features/motorista/captura-bdo`), adicionando apenas o subdomínio `bagagemcarona`/`bagagem-carona` — não justifica um domínio de negócio próprio, dado o baixo volume de regras (FR-001 a FR-005).

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
