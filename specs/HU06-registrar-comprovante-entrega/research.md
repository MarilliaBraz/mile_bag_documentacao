# Fase 0 — Pesquisa e Decisões Técnicas

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

As decisões de armazenamento de imagem, isolamento multi-tenant e operação offline aplicáveis a esta HU já estão registradas em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md) (seções 3, 6 e 8). Não há stack nova a decidir para esta HU.

## 1. Tratamento de falha na captura de geolocalização

**Decision**: Se a geolocalização não puder ser obtida no momento do registro, o restante do comprovante é retido localmente (rascunho) e a interface solicita nova tentativa de captura de localização antes de permitir a confirmação final — o comprovante não é aceito sem geolocalização (mantendo FR-001), mas o motorista não perde o restante dos dados já preenchidos.

**Rationale**: A geolocalização é um dos campos obrigatórios do comprovante (RD06); descartá-la silenciosamente comprometeria a força probatória do registro perante a companhia aérea. Reter o rascunho evita que uma falha momentânea de GPS obrigue o motorista a preencher tudo de novo.

**Alternatives considered**: Aceitar o comprovante sem geolocalização e sinalizar como "pendente" foi descartado — violaria RN04/RD06, que definem a geolocalização como parte do comprovante, não como campo opcional.
