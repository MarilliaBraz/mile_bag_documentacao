# Implementation Plan: Registrar ocorrência de não entrega

**Branch**: `HU07-registrar-ocorrencia-nao-entrega` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU07-registrar-ocorrencia-nao-entrega/spec.md`

## Summary

Permite ao motorista registrar, a partir de um catálogo padronizado por tenant, o motivo pelo qual uma entrega não pôde ser concluída, reabrindo o BDO automaticamente e preservando o histórico de tentativas. Reaproveita a stack definida em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md); depende do catálogo configurável descrito em HU11.

## Technical Context

Herda o Technical Context do plano-mãe. Não há dependência técnica nova; o catálogo de motivos de ocorrência é dado de configuração por tenant (não hardcoded), consistente com o pilar de "parametrização" de `CONVENTIONS.md` §2.3.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Avaliação para esta HU |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — histórico de ocorrências nunca é sobrescrito (FR-004) |
| Os 3 pilares: parametrização | PASS — motivo de ocorrência vem de catálogo configurável por tenant, não de enum fixo no código |
| Separação `core`/`business` (backend) | PASS — `business/ocorrencia` depende de `business/entrega` e `business/bdo` (reabertura), sem acoplar lógica genérica em `core` |
| Organização por `feature` (frontend) | PASS — UI entra em `features/motorista/ocorrencia`; o cadastro do catálogo em si pertence a `features/back-office/configuracao-tenant` (HU11) |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU07-registrar-ocorrencia-nao-entrega/
├── spec.md
├── plan.md          # este arquivo
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/
├── controller/     # endpoint de registro de ocorrência e consulta do catálogo de motivos
├── service/        # reabertura do BDO, transição de estado da entrega
├── repository/
└── model/          # entidades Ocorrencia, MotivoOcorrencia

mile_bag_app/src/features/motorista/ocorrencia/
└── (seleção de motivo, descrição opcional, envio ao backend ou fila offline)
```

**Structure Decision**: Domínio `ocorrencia` próprio no backend, consumindo `business/entrega` (para transição de estado) e o catálogo de motivos gerido por `business/configuracao` (HU11) — sem duplicar a lógica de parametrização por tenant.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
