# Implementation Plan: HU02 — Autoatribuir a entrega

**Branch**: `HU02-autoatribuir-entrega` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU02-autoatribuir-entrega/spec.md`

## Summary

Atribuição do motorista responsável por uma entrega, disparada logo após a confirmação do BDO (HU01). Segunda etapa do ciclo de entrega (User Story 1 do spec guarda-chuva).

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md). Específico desta HU:

- **Back-end** (`mile_bag_api`): pacote `business/entrega`, criação do registro de Entrega a partir de um BDO confirmado e associação ao motorista autenticado (via claim do JWT).
- **Front-end** (`mile_bag_app`, PWA motorista): ação de autoatribuição na feature `motorista/captura-bdo` (continuação do fluxo de HU01) e listagem de entregas em andamento.
- Requer controle de concorrência (impedir dupla atribuição) — tratado como constraint de unicidade no back-end, não como decisão de stack nova.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates detalhados no plano-mãe. Para esta HU:

| Gate | Avaliação |
|---|---|
| Separação `core`/`business` (backend) | PASS — regra de atribuição em `business/entrega`, sem lógica de negócio em `core` |
| Segurança e privacidade por padrão | PASS — motorista responsável é sempre o usuário autenticado (JWT), nunca um parâmetro livre vindo do cliente |

Nenhuma violação. Complexity Tracking não se aplica.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU02-autoatribuir-entrega/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/
├── controller/     # endpoint de autoatribuição e listagem
├── service/        # regra de unicidade do motorista responsável
├── repository/
└── model/          # entidade Entrega

mile_bag_app/src/features/motorista/captura-bdo/
└── (ação de autoatribuição encadeada à confirmação do BDO — HU01)
```

**Structure Decision**: Introduz o pacote `business/entrega` no backend, reaproveitado por todas as demais HUs do ciclo de entrega (HU03–HU08). No frontend, a ação vive na mesma feature de captura do BDO, por ser um passo sequencial do mesmo fluxo.
