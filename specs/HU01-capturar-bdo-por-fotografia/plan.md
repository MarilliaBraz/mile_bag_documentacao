# Implementation Plan: HU01 — Capturar o BDO por fotografia

**Branch**: `HU01-capturar-bdo-por-fotografia` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU01-capturar-bdo-por-fotografia/spec.md`

## Summary

Captura fotográfica do BDO pelo motorista, com extração automática de campos por OCR e confirmação obrigatória em tela de conferência. Primeira etapa do ciclo de entrega (User Story 1 do spec guarda-chuva).

## Technical Context

Stack completa e racionalidade detalhadas em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md). Específico desta HU:

- **Back-end** (`mile_bag_api`): endpoint de upload de imagem + OCR (Tesseract 5 via Tess4J), pacote `business/bdo`.
- **Front-end** (`mile_bag_app`, PWA motorista): captura de câmera e tela de conferência, feature `motorista/captura-bdo`.
- **Storage**: imagem do BDO em volume de disco com prefixo por tenant (ver plano-mãe, seção Storage).
- Sem itens de tecnologia específicos desta HU além dos já decididos no plano-mãe.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates avaliados em detalhe no plano-mãe (`../001-milebag-user-stories/plan.md`, seção Constitution Check), com base em `mile_bag_audite/CONSTITUTION.md` e `CONVENTIONS.md`. Para esta HU:

| Gate | Avaliação |
|---|---|
| Separação `core`/`business` (backend) | PASS — OCR e regras de conferência ficam em `business/bdo`, sem lógica de negócio em `core` |
| Organização por `feature` (frontend) | PASS — captura e conferência isoladas em `features/motorista/captura-bdo` |
| Confiabilidade/rastreabilidade dos dados | PASS — imagem original preservada (FR-006), conferência obrigatória (FR-003, FR-005) |

Nenhuma violação. Complexity Tracking não se aplica.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU01-capturar-bdo-por-fotografia/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/
├── controller/     # endpoint de upload e conferência
├── service/        # orquestração do OCR (Tess4J) e persistência
├── repository/
├── model/          # entidade BDO, DTOs de extração
└── mapper/

mile_bag_app/src/features/motorista/captura-bdo/
├── components/     # captura de câmera, tela de conferência
├── hooks/
├── services/        # chamada à API de upload/conferência
└── types/
```

**Structure Decision**: Reaproveita a estrutura `core`/`business` e `features` definida no plano-mãe; nenhum pacote novo além de `business/bdo` (backend) e `features/motorista/captura-bdo` (frontend).
