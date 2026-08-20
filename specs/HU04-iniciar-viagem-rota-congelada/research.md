# Fase 0 — Pesquisa e Decisões Técnicas: HU04

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A decisão central desta HU — tratar o snapshot de rota como um registro imutável, próprio, persistido no início da viagem — já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção 7 ("Congelamento (snapshot) da rota"), incluindo a rationale (RN02/RD04 de `../../requisitos/`) e a alternativa descartada (recalcular sob demanda).

**Ponto específico desta HU**: a imutabilidade (FR-003) deve ser garantida na camada de persistência (ex.: sem operação de `update` exposta para os campos do snapshot após a criação), e não apenas por convenção de código na camada de serviço — para que a garantia sobreviva a qualquer novo desenvolvedor que venha a mexer no pacote `business/viagem` no futuro.
