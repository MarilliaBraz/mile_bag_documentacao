# Fase 0 — Pesquisa e Decisões Técnicas: HU03

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A decisão de motor de rotas/geocodificação (Valhalla ou OSRM; Nominatim ou Photon) já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção 5, incluindo a recomendação de Valhalla como default por suportar melhor múltiplas rotas alternativas — requisito direto de FR-002 desta HU.

**Ponto específico desta HU**: geocodificação de endereço com falha (Edge Case do spec) exige que a interface permita reposicionar manualmente o pino do endereço de entrega no mapa antes de recalcular a rota — comportamento de UI a implementar na feature `motorista/rota`, sem impacto na escolha do motor de rotas.
