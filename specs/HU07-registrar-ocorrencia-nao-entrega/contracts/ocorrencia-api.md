# Contrato de API — Ocorrência

**Feature**: [../spec.md](../spec.md)

## `GET /api/v1/tenants/{tenantId}/motivos-ocorrencia`

Lista o catálogo de motivos de ocorrência ativos do tenant, para preenchimento da tela de registro.

**Response 200 OK**:
```
[
  { "id": "<id>", "nome": "Ausência do recebedor" },
  { "id": "<id>", "nome": "Endereço não localizado" }
]
```

## `POST /api/v1/entregas/{entregaId}/ocorrencias`

Registra uma ocorrência para a entrega e reabre o BDO correspondente.

**Request**:
- `entregaId` (path).
- Body: `{ "motivoOcorrenciaId": "<id>", "descricaoAdicional": "<texto opcional>", "idLocal": "<id gerado no dispositivo, para sincronização offline>" }`

**Response 201 Created**:
```
{
  "id": "<id da ocorrência>",
  "entregaId": "<entregaId>",
  "motivoOcorrenciaId": "<id>",
  "dataHora": "<timestamp>",
  "entregaStatus": "REABERTA"
}
```

**Erros**:
- `404 Not Found` — `entregaId` inexistente, ou `motivoOcorrenciaId` não pertence ao catálogo do tenant da entrega.
- `409 Conflict` — a entrega já está encerrada com comprovante de entrega (FR-003).

## `GET /api/v1/entregas/{entregaId}/ocorrencias`

Lista o histórico de ocorrências de uma entrega, em ordem cronológica (FR-004).

**Response 200 OK**: array de objetos no mesmo formato do `POST` acima.
