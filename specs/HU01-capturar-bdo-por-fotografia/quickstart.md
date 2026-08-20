# Quickstart de Validação: HU01 — Capturar o BDO por fotografia

**Feature**: [spec.md](./spec.md) | **Contratos**: [contracts/bdo-api.md](./contracts/bdo-api.md) | **Modelo de dados**: [data-model.md](./data-model.md)

## Pré-requisitos

- Motorista autenticado, associado a um tenant já provisionado.
- Ambiente com o serviço de OCR (Tesseract) disponível (ver `../001-milebag-user-stories/plan.md`, Target Platform).
- Um BDO real ou de teste, no padrão de telex WorldTracer, disponível para fotografar.

## Passos

1. Autenticar como motorista no aplicativo.
2. Fotografar o BDO de teste (`POST /api/v1/bdos`).
3. Verificar que a tela de conferência exibe os campos extraídos (comparar com o documento original).
4. Corrigir manualmente qualquer campo extraído incorretamente.
5. Confirmar o registro (`PATCH /api/v1/bdos/{id}/conferencia`).
6. Repetir a fotografia do mesmo BDO e verificar a sinalização de duplicidade (`GET /api/v1/bdos?possivelDuplicidade=true`).

## Resultado esperado

- O BDO aparece com `statusConferencia = confirmado` somente após o passo 5 — nunca antes.
- A imagem original permanece acessível no registro após a confirmação.
- Na repetição do passo 6, o sistema sinaliza a duplicidade antes de permitir um novo registro.
- Critério de sucesso de referência: acerto de extração ≥ 90% dos campos obrigatórios (SC-001 do spec.md desta HU).
