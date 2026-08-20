# Fase 0 — Pesquisa e Decisões Técnicas: HU02

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Não há decisões de tecnologia novas nesta HU além das já registradas em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md) (seção 3, isolamento multi-tenant e propagação da identidade do usuário via JWT).

**Ponto específico desta HU (padrão de implementação, não escolha de tecnologia)**: a unicidade do motorista responsável (FR-004 do spec desta HU) deve ser garantida por uma constraint no banco de dados (não apenas checagem em memória na aplicação), para evitar condição de corrida no cenário de dois motoristas autoatribuindo a mesma entrega quase simultaneamente (Edge Case do spec).
