# Fase 0 — Pesquisa e Decisões Técnicas

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A maior parte das decisões técnicas relevantes a esta HU já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), em especial as seções 3 (isolamento multi-tenant), 6 (operação offline) e 7 (congelamento de rota) — a captura da bagagem carona reaproveita o mesmo fluxo de captura de BDO e a mesma viagem/snapshot já em execução.

## 1. Cálculo da quilometragem complementar

**Decision**: A quilometragem complementar de uma bagagem carona é o trecho adicional de deslocamento necessário para incluir o endereço da bagagem carona na rota já em execução, calculado no momento da vinculação e congelado junto ao registro da bagagem carona (mesmo princípio de imutabilidade do snapshot de rota, seção 7 do research-mãe).

**Rationale**: RN03 exige que a remuneração considere "apenas a quilometragem complementar", não uma viagem completa — isso só é auditável se o valor for calculado e congelado no momento do vínculo, e não recalculado depois.

**Alternatives considered**: Tratar a bagagem carona como uma fração fixa da tarifa da entrega principal foi descartado — não reflete o esforço real de deslocamento e diverge do texto da fonte, que fala explicitamente em "quilometragem complementar".

Nenhuma outra decisão técnica nova é necessária para esta HU.
