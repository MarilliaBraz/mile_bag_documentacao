# Implementation Plan: HU03 — Calcular e escolher a rota da entrega

**Branch**: `HU03-calcular-e-escolher-rota` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU03-calcular-e-escolher-rota/spec.md`

## Summary

Cálculo de rotas alternativas até o endereço de entrega e seleção pelo motorista (sugerida ou própria). Terceira etapa do ciclo de entrega (User Story 1 do spec guarda-chuva); antecede o congelamento da viagem (HU04).

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md). Específico desta HU:

- **Back-end** (`mile_bag_api`): pacote `business/rota`, cliente para o serviço de rotas open source (Valhalla ou OSRM) e geocodificação (Nominatim ou Photon) — ver decisão em `../001-milebag-user-stories/research.md`, seção 5.
- **Front-end** (`mile_bag_app`, PWA motorista): feature `motorista/rota`, exibição das alternativas em mapa (Leaflet sobre OpenStreetMap) e captura do traçado manual quando o motorista opta por rota própria.
- Atribuição visível do OpenStreetMap na tela de mapa é obrigatória (RI08, licença ODbL).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates detalhados no plano-mãe. Para esta HU:

| Gate | Avaliação |
|---|---|
| Separação `core`/`business` (backend) | PASS — integração com o serviço de rotas fica em `business/rota`; nada de específico de rotas em `core` |
| Organização por `feature` (frontend) | PASS — `features/motorista/rota` isolada, reutiliza componente de mapa de `shared/` |
| Simplicidade antes de generalidade | PASS — nenhuma otimização de múltiplas paradas ou roteirização automática é implementada (fora de escopo do produto, `requisitos/03-regras-negocio.md` RN12) |

Nenhuma violação. Complexity Tracking não se aplica.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU03-calcular-e-escolher-rota/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/rota/
├── controller/     # endpoint de cálculo de rota e seleção
├── service/        # integração com Valhalla/OSRM e Nominatim/Photon
├── repository/
└── model/          # DTOs de rota proposta (não confundir com o snapshot congelado de HU04)

mile_bag_app/src/features/motorista/rota/
├── components/     # mapa (Leaflet), lista de rotas alternativas, traçado manual
├── hooks/
├── services/
└── types/
```

**Structure Decision**: O pacote `business/rota` fica responsável apenas pelas propostas de rota (não confundir com a Viagem/snapshot, tratada em HU04). O componente de mapa (`shared/mapa`) é reutilizado por esta HU e por HU04.
