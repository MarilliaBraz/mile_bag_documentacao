# Quickstart — Registrar ocorrência de não entrega

**Feature**: [spec.md](./spec.md) | **Modelo de dados**: [data-model.md](./data-model.md) | **Contratos**: [contracts/ocorrencia-api.md](./contracts/ocorrencia-api.md)

## Pré-requisitos

- Um tenant com ao menos um motivo de ocorrência cadastrado (HU11).
- Uma entrega em andamento (viagem iniciada, HU04).

## Passos

1. Chamar `GET /api/v1/tenants/{tenantId}/motivos-ocorrencia` e confirmar que ao menos um motivo é retornado.
2. Chamar `POST /api/v1/entregas/{entregaId}/ocorrencias` com um `motivoOcorrenciaId` válido — confirmar `201 Created` e `entregaStatus: "REABERTA"`.
3. Chamar `GET /api/v1/entregas/{entregaId}/ocorrencias` e confirmar que a ocorrência registrada aparece no histórico.
4. Registrar o comprovante de entrega para a mesma entrega (HU06) e, em seguida, tentar registrar uma nova ocorrência — confirmar `409 Conflict` (FR-003).
5. Consultar novamente `GET /api/v1/entregas/{entregaId}/ocorrencias` e confirmar que a ocorrência do passo 2 continua no histórico, mesmo após a entrega ter sido concluída com sucesso (FR-004).

## Resultado esperado

- Toda entrega malsucedida resulta em uma ocorrência registrada e num BDO reaberto, sem intervenção manual do administrador do tenant (SC-002).
- O histórico de tentativas de uma entrega permanece completo e consultável (SC-003).
