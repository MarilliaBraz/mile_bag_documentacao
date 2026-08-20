# Implementation Plan: Registrar o comprovante de entrega

**Branch**: `HU06-registrar-comprovante-entrega` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU06-registrar-comprovante-entrega/spec.md`

## Summary

Encerra o ciclo de uma entrega mediante o registro do comprovante (foto do documento assinado, recebedor, data, hora e geolocalização), sempre passível de captura offline. Reaproveita integralmente a stack definida em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md).

## Technical Context

Herda o Technical Context do plano-mãe. Ponto específico desta HU: a foto do comprovante e a geolocalização exigem acesso à câmera e ao GPS do dispositivo do motorista — já previsto na PWA (`Workbox`/armazenamento local) descrita no plano-mãe.

**Storage específico**: a imagem do comprovante segue o mesmo padrão de armazenamento por tenant descrito no plano-mãe (volume de disco com prefixo por tenant), aplicado tanto à imagem do BDO quanto à do comprovante.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Avaliação para esta HU |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — o comprovante é a prova documental exigida pela companhia aérea; nenhum campo opcional substitui os obrigatórios (FR-001) |
| Segurança e privacidade por padrão | PASS — dados do recebedor (nome) e geolocalização tratados como dados pessoais sob LGPD, mesmo enquadramento do plano-mãe |
| Separação `core`/`business` (backend) | PASS — lógica de encerramento de entrega entra em `business/comprovante`, dependente de `business/entrega` |
| Organização por `feature` (frontend) | PASS — UI entra em `features/motorista/comprovante-entrega` |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU06-registrar-comprovante-entrega/
├── spec.md
├── plan.md          # este arquivo
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/
├── controller/     # endpoint de registro do comprovante
├── service/        # transição de estado da entrega, upload de imagem
├── repository/
└── model/          # entidade ComprovanteEntrega

mile_bag_app/src/features/motorista/comprovante-entrega/
└── (captura de foto, formulário de recebedor, geolocalização, envio ao backend ou fila offline)
```

**Structure Decision**: Domínio próprio `comprovante` no backend (separado de `entrega`, pois tem regra de validação e upload de imagem específicos), consistente com a separação por domínio já adotada no plano-mãe.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
