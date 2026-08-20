# Fase 0 — Pesquisa e Decisões Técnicas: HU10

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A decisão de motor de rotas (Valhalla ou OSRM sobre OpenStreetMap) já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção **"5. Cálculo de rota, distância e pedágios"**. Esta HU não introduz nova tecnologia — apenas o cadastro e o versionamento de valores que alimentam esse cálculo.

## Versionamento de valor com data de vigência

**Decision**: Cada praça de pedágio tem uma lista de valores históricos, cada um com sua própria `vigenteDesde`; o valor aplicável a uma viagem é o de maior `vigenteDesde` que seja `<=` à data da viagem. Nenhum valor é atualizado in-place — todo novo valor é uma nova linha.

**Rationale**: Requisito de negócio explícito (RN02, RD08, FR-002/FR-003 do spec.md desta HU): snapshots de viagens já congeladas não podem ser afetados por atualizações de cadastro posteriores. Um histórico append-only é a forma mais simples de garantir isso sem duplicar o valor de pedágio dentro de cada snapshot de viagem no momento do cadastro (o snapshot já guarda o valor resolvido, não uma referência à praça — ver [data-model do spec-mãe](../001-milebag-user-stories/spec.md#key-entities), entidade "Viagem / snapshot de rota").

**Alternatives considered**: Atualizar o valor in-place e confiar apenas no snapshot da viagem para preservar o histórico foi descartado — perderia a rastreabilidade de "qual era o valor cadastrado em tal data", útil para auditoria e para o Edge Case de rejeitar vigências retroativas (FR-005).
