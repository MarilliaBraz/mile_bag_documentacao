# Fase 0 — Pesquisa e Decisões Técnicas: HU15

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Não coberto em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md) — esta HU introduz dois pontos novos.

## 1. Provedor de envio de e-mail transacional

**Decision**: Usar um serviço SMTP já disponível gratuitamente (ex.: conta de e-mail transacional gratuita ou SMTP relay de baixo volume), configurado via Spring Boot Mail, sem dependência de um provedor comercial pago (ex. SendGrid, SES pago).

**Rationale**: Consistente com a restrição de custo zero do piloto (RNF08 do spec-mãe) — todo o restante da pilha já evita serviços pagos. O volume de convites no piloto é baixo (um por tenant cadastrado), compatível com camadas gratuitas.

**Alternatives considered**: Provedores comerciais de e-mail transacional foram descartados para o piloto pela mesma restrição de custo; podem ser reavaliados na expansão comercial, fora do escopo deste piloto (ver `evidencias/Projeto_de_software_-_ADS_com_TAP.md`, seção 3.12, "Passam a existir custos apenas na eventual expansão comercial").

## 2. Token de convite

**Decision**: Token opaco, gerado aleatoriamente (não previsível/sequencial), de uso único, associado a um prazo de expiração e ao `tenantId` de destino; invalidado tanto no uso quanto no reenvio de um novo convite para o mesmo tenant (FR-004, FR-006).

**Rationale**: Requisito de segurança explícito do spec — um token previsível ou reutilizável permitiria a um terceiro assumir indevidamente a administração de um tenant.

**Alternatives considered**: Usar o próprio e-mail do destinatário como identificador do convite foi descartado — não impede reuso nem oferece expiração natural sem lógica adicional equivalente à de um token dedicado.
