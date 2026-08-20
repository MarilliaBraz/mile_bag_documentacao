# Implementation Plan: HU09 — Configurar tabelas de tarifa

**Branch**: `HU09-configurar-tabelas-tarifa` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU09-configurar-tabelas-tarifa/spec.md`

## Summary

CRUD de tarifas por tenant (padrão, por motorista, por região), com resolução de especificidade no momento do fechamento. Faz parte do domínio `tarifa` já mapeado no [plano-mãe](../001-milebag-user-stories/plan.md).

## Technical Context

Stack completa (linguagens, frameworks, banco, isolamento multi-tenant, contêineres) já definida no [Technical Context do plano-mãe](../001-milebag-user-stories/plan.md#technical-context) — não repetida aqui. Específico desta HU: nenhuma dependência nova; é CRUD simples sobre PostgreSQL via Spring Data JPA, sem OCR, rotas ou processamento assíncrono.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Mesmos gates avaliados no [plano-mãe](../001-milebag-user-stories/plan.md#constitution-check), reavaliados para esta HU:

| Gate | Avaliação para HU09 |
|---|---|
| Separação `core`/`business` (backend) | PASS — tarifa é um subpacote `business/tarifa` autocontido, sem lógica de negócio vazando para `core` |
| Isolamento entre tenants (RNF01) | PASS — toda tarifa carrega a coluna de tenant e está sujeita à mesma política de Row Level Security do restante do domínio |
| Parametrização (3 pilares) | PASS — é a própria funcionalidade que implementa a parametrização de valores de negócio (tarifas), não hardcoded |

Nenhuma violação identificada.

## Project Structure

Caminhos relevantes dentro dos repositórios já descritos no [plano-mãe](../001-milebag-user-stories/plan.md#project-structure):

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/
├── controller/     # endpoints de CRUD de tarifa (ver contracts/)
├── service/        # resolução de especificidade (motorista > região > padrão)
├── repository/
└── model/          # TabelaTarifa (ver data-model.md)

mile_bag_app/src/features/back-office/tarifas/    # tela de cadastro/listagem de tarifas (HU09)
```

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
