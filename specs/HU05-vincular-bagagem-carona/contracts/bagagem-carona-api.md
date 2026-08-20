# Contrato de API — Bagagem Carona

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/entregas/{entregaId}/bagagens-carona`

Vincula uma bagagem carona (a partir de um BDO já capturado) a uma entrega principal em andamento.

**Request**:
- `entregaId` (path) — identificador da entrega principal.
- Body: `{ "bdoId": "<id do BDO da bagagem carona>" }`

**Response 201 Created**:
```
{
  "id": "<id da bagagem carona>",
  "entregaPrincipalId": "<entregaId>",
  "bdoId": "<bdoId>",
  "quilometragemComplementar": <decimal>,
  "criadoEm": "<timestamp>"
}
```

**Erros**:
- `404 Not Found` — `entregaId` ou `bdoId` inexistente (ou pertencente a outro tenant — nunca revelado como "outro tenant", apenas como não encontrado).
- `409 Conflict` — entrega principal já encerrada com comprovante (FR-002), ou `bdoId` já vinculado a outra bagagem carona (FR-003).

## `DELETE /api/v1/entregas/{entregaId}/bagagens-carona/{bagagemCaronaId}`

Remove o vínculo de uma bagagem carona, enquanto a entrega principal ainda não tiver sido encerrada com comprovante (edge case de desvínculo por engano).

**Response 204 No Content**.

**Erros**:
- `404 Not Found` — vínculo inexistente.
- `409 Conflict` — entrega principal já encerrada com comprovante.

## `GET /api/v1/entregas/{entregaId}/bagagens-carona`

Lista as bagagens carona vinculadas a uma entrega principal (usado pela tela do motorista e pelo fechamento periódico).

**Response 200 OK**: array de objetos no mesmo formato do `POST` acima.
