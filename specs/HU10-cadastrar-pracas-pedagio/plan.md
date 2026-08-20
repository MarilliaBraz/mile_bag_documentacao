# Implementation Plan: HU10 — Cadastrar praças de pedágio

**Branch**: `HU10-cadastrar-pracas-pedagio` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU10-cadastrar-pracas-pedagio/spec.md`

## Summary

Cadastro de praças de pedágio por tenant, com histórico de valores versionado por data de vigência (nunca sobrescrito), consumido no congelamento de rota e no cálculo de reembolso (sempre ida + volta). Faz parte do domínio `pedagio` já mapeado no [plano-mãe](../001-milebag-user-stories/plan.md).

## Technical Context

Stack completa já definida no [Technical Context do plano-mãe](../001-milebag-user-stories/plan.md#technical-context) — não repetida aqui. Específico desta HU: o motor de rotas (Valhalla/OSRM) identifica os trechos com pedágio de uma rota calculada; a associação desses trechos às praças cadastradas (por nome/localização) é lógica de aplicação, não do motor de rotas em si — ver [research.md](./research.md).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Mesmos gates avaliados no [plano-mãe](../001-milebag-user-stories/plan.md#constitution-check), reavaliados para esta HU:

| Gate | Avaliação para HU10 |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — histórico de valores com vigência nunca sobrescrito (FR-002), mesmo princípio do snapshot imutável de rota |
| Separação `core`/`business` (backend) | PASS — subpacote `business/pedagio` autocontido |
| Isolamento entre tenants (RNF01) | PASS — praça de pedágio carrega coluna de tenant, sujeita à mesma RLS do restante do domínio |

Nenhuma violação identificada.

## Project Structure

Caminhos relevantes dentro dos repositórios já descritos no [plano-mãe](../001-milebag-user-stories/plan.md#project-structure):

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/
├── controller/     # endpoints de CRUD de praça de pedágio (ver contracts/)
├── service/        # resolução do valor vigente em uma data; cálculo de reembolso (ida+volta)
├── repository/
└── model/          # PracaPedagio, PracaPedagioValor (ver data-model.md)

mile_bag_app/src/features/back-office/pedagios/    # tela de cadastro/listagem de praças (HU10)
```

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
