# Fase 0 — Pesquisa e Decisões Técnicas: HU16

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Não há decisão de stack nova além do que já está em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md) — `PlanoAssinatura` é uma entidade transacional comum, sem integração externa. Único ponto a esclarecer:

## Sinalização de excedente ao trocar de plano

**Decision**: A troca de plano de um tenant é sempre permitida (não bloqueada por excedente de uso atual); o sistema registra e expõe um indicador de "uso acima do limite do plano vigente" para o tenant, consultável pelo administrador da plataforma e pelo administrador do tenant.

**Rationale**: Bloquear a troca de plano criaria um impasse operacional (um tenant que já excedeu o limite anterior nunca poderia migrar para regularizar sua situação). A aplicação efetiva do limite (impedir novos motoristas/BDOs além do limite) já é responsabilidade de FR-017 do spec-mãe, executada nas HUs que criam esses recursos — esta HU só cuida da sinalização no momento da troca (FR-006, Edge Case 1).

**Alternatives considered**: Bloquear a troca até o tenant reduzir o uso foi descartado por ser operacionalmente inviável (o excedente normalmente só se resolve com a mudança de plano, não antes dela).
