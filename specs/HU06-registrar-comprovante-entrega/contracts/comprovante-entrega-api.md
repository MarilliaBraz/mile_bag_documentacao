# Contrato de API — Comprovante de Entrega

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/entregas/{entregaId}/comprovante`

Registra o comprovante de entrega (POD) e encerra a entrega com sucesso.

**Request**:
- `entregaId` (path) — identificador da entrega.
- Body (`multipart/form-data`):
  - `foto` — arquivo de imagem do documento assinado (obrigatório)
  - `identificacaoRecebedor` — texto (obrigatório)
  - `dataHora` — timestamp ISO 8601, capturado no dispositivo (obrigatório)
  - `latitude`, `longitude` — decimais (obrigatórios)
  - `idLocal` — identificador gerado no dispositivo, usado como chave de idempotência quando o registro vier de sincronização offline (ver contrato de HU08)

**Response 201 Created**:
```
{
  "id": "<id do comprovante>",
  "entregaId": "<entregaId>",
  "dataHora": "<timestamp>",
  "entregaStatus": "ENCERRADA_COM_SUCESSO"
}
```

**Erros**:
- `422 Unprocessable Entity` — algum campo obrigatório ausente (FR-002).
- `409 Conflict` — a entrega já possui um comprovante registrado (FR-004).
- `404 Not Found` — `entregaId` inexistente (ou de outro tenant).
