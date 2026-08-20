# Contrato de API: Rota (HU03)

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/entregas/{entregaId}/rotas/calcular`

Calcula as alternativas de rota entre a base do motorista e o endereço de entrega do BDO associado.

- **Request**: opcionalmente, coordenadas de destino ajustadas manualmente (quando a geocodificação automática falhou).
- **Response 200**: lista de alternativas de rota (distância, tempo estimado, trechos com pedágio).
- **Erros**:
  - `404` — entrega não encontrada, não pertence ao tenant, ou ainda não autoatribuída.
  - `422` — endereço não pôde ser geocodificado e nenhuma coordenada manual foi informada.

## `POST /api/v1/entregas/{entregaId}/rotas/selecionar`

Registra a rota escolhida pelo motorista (sugerida ou própria), como preparação para o início da viagem (HU04).

- **Request**: `tipoSelecao` (`sugerida` ou `propria`), identificador da alternativa (quando sugerida) ou traçado manual + `justificativa` (quando própria).
- **Response 200**: seleção registrada, pronta para ser congelada por `POST /api/v1/entregas/{entregaId}/viagem/iniciar` (contrato de HU04).
- **Erros**:
  - `400` — `tipoSelecao = propria` sem `justificativa`.
  - `404` — entrega não encontrada ou não pertence ao tenant.
