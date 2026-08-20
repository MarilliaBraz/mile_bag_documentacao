# Implementation Plan: HU04 — Iniciar a viagem com a rota congelada

**Branch**: `HU04-iniciar-viagem-rota-congelada` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU04-iniciar-viagem-rota-congelada/spec.md`

## Summary

Congelamento, no início da viagem, do snapshot imutável de rota (quilometragem, pedágios, tempo estimado) que servirá de base exclusiva para o cálculo do pagamento. Quarta e última etapa do núcleo de execução da entrega antes do comprovante (HU06).

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md). Específico desta HU:

- **Back-end** (`mile_bag_api`): pacote próprio `business/viagem` — decisão: IRMÃO de `business/rota` (HU03), não aninhado sob `business/entrega`. Motivo: `Rota` (cálculo transitório, HU03) e `Viagem` (snapshot persistido, HU04) são o "antes e depois" do mesmo conceito, o que os agrupa naturalmente entre si; além disso `Viagem` é consumida diretamente por outras features (ex.: HU12, fechamento) sem precisar navegar por `Entrega`. Persiste o snapshot como *value object* imutável — ver decisão e racionalidade em `../001-milebag-user-stories/research.md`, seção 7 ("Congelamento (snapshot) da rota").
- **Front-end** (`mile_bag_app`, PWA motorista): ação "iniciar viagem" na feature `motorista/rota`, encadeada à seleção feita em HU03.
- Nenhuma decisão de stack nova; a única exigência técnica é a imutabilidade do registro após gravado (ver research desta HU).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates detalhados no plano-mãe. Para esta HU:

| Gate | Avaliação |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — é exatamente o gate que esta HU implementa (`CONSTITUTION.md` §2.1); nenhuma alteração silenciosa do snapshot é permitida (FR-003) |
| Separação `core`/`business` (backend) | PASS — persistência do snapshot fica em `business/viagem`, sem lógica de negócio em `core` |

Nenhuma violação. Complexity Tracking não se aplica.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU04-iniciar-viagem-rota-congelada/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/
├── controller/     # endpoint de início de viagem
├── service/        # regra de congelamento e imutabilidade do snapshot
├── repository/
└── model/          # entidade Viagem (snapshot imutável)

mile_bag_app/src/features/motorista/rota/
└── (ação "iniciar viagem", encadeada à seleção de rota de HU03)
```

**Structure Decision**: O snapshot é modelado como entidade própria (`Viagem`), separada da Proposta de Rota transitória de HU03, justamente para deixar explícita a fronteira de imutabilidade descrita no research desta HU.
