# Fase 1 — Modelo de Dados: HU03

**Feature**: [spec.md](./spec.md)

## Entidade: Proposta de Rota (transitória)

Representa uma opção de trajeto calculada para uma entrega já autoatribuída (ver `../HU02-autoatribuir-entrega/data-model.md`). Não é persistida como registro definitivo — existe apenas até a seleção do motorista, que então alimenta o snapshot imutável de HU04 (`../HU04-iniciar-viagem-rota-congelada/data-model.md`).

| Campo | Descrição | Regra |
|---|---|---|
| entregaId | Entrega para a qual a rota foi calculada | Obrigatório |
| origem | Ponto de partida (base do motorista) | — |
| destino | Endereço de entrega (geocodificado a partir do BDO, ou ajustado manualmente) | Editável manualmente quando a geocodificação falha (FR-006) |
| alternativas | Lista de opções de trajeto, cada uma com distância, tempo estimado e trechos com pedágio | Ao menos uma opção quando o cálculo automático é possível |
| tipoSelecao | `sugerida` \| `propria` | Determina se `justificativa` é obrigatória |
| justificativa | Texto livre exigido quando `tipoSelecao = propria` | Obrigatório nesse caso (FR-005) |

**Validações**:
- `justificativa` MUST estar preenchida quando `tipoSelecao = propria`.
- A Proposta de Rota não gera nenhum efeito no cálculo de pagamento por si só — isso só ocorre quando ela é congelada em uma Viagem (HU04).

**Relacionamentos**: uma Proposta de Rota referencia uma Entrega (HU02) e, quando selecionada e a viagem é iniciada, origina o snapshot de Viagem (HU04).
