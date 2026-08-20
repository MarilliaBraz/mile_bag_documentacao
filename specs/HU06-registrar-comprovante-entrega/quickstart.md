# Quickstart — Registrar o comprovante de entrega

**Feature**: [spec.md](./spec.md) | **Modelo de dados**: [data-model.md](./data-model.md) | **Contratos**: [contracts/comprovante-entrega-api.md](./contracts/comprovante-entrega-api.md)

## Pré-requisitos

- Uma entrega em andamento, com viagem já iniciada (HU04).

## Passos

1. Chamar `POST /api/v1/entregas/{entregaId}/comprovante` sem o campo `identificacaoRecebedor` — confirmar `422 Unprocessable Entity` (FR-002).
2. Repetir a chamada com todos os campos obrigatórios preenchidos — confirmar `201 Created` e que a resposta traz `entregaStatus: "ENCERRADA_COM_SUCESSO"`.
3. Repetir a mesma chamada para a mesma entrega — confirmar `409 Conflict` (FR-004).
4. Simular o dispositivo do motorista offline (ver quickstart de HU08) e repetir o passo 2 — confirmar que o registro é aceito localmente e sincronizado depois, resultando no mesmo estado final.

## Resultado esperado

- A entrega consultada via API reflete o estado "encerrada com sucesso" apenas após o comprovante válido ser registrado (SC-002).
- O comprovante registrado offline chega ao servidor com os mesmos dados capturados no dispositivo, sem exigir reentrada manual (SC-003).
