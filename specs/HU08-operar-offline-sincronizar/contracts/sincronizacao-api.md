# Contrato de API — Sincronização Offline

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/sync/eventos`

Envia, em lote, os eventos acumulados offline no dispositivo do motorista, para processamento ordenado e idempotente pelo servidor.

**Request**:
```
{
  "eventos": [
    {
      "idLocal": "<uuid gerado no dispositivo>",
      "sequenciaLocal": <inteiro>,
      "tipoEvento": "BDO_CAPTURADO" | "ENTREGA_AUTOATRIBUIDA" | "ROTA_ESCOLHIDA" |
                    "VIAGEM_INICIADA" | "BAGAGEM_CARONA_VINCULADA" |
                    "COMPROVANTE_REGISTRADO" | "OCORRENCIA_REGISTRADA",
      "timestampCriacaoLocal": "<timestamp ISO 8601>",
      "payload": { /* mesmo formato do corpo do endpoint síncrono equivalente (HU01–HU07) */ }
    }
  ]
}
```

O array `eventos` MUST estar ordenado por `sequenciaLocal` crescente (FR-005).

**Response 200 OK**:
```
{
  "resultados": [
    { "idLocal": "<uuid>", "resultado": "ACEITO", "idServidor": "<id do registro criado>" },
    { "idLocal": "<uuid>", "resultado": "DUPLICADO" },
    { "idLocal": "<uuid>", "resultado": "ERRO", "motivo": "<descrição>" }
  ]
}
```

Processamento é *melhor esforço item a item* (ver `research.md`, seção 4): uma falha em um item não interrompe o processamento dos demais. O status HTTP da resposta é sempre `200 OK` quando o lote é recebido e processado — os resultados individuais em `resultados` informam o que aconteceu com cada evento; o cliente reenvia apenas os itens com `resultado: "ERRO"` em uma tentativa futura.

**Erros de nível de requisição** (aplicam-se ao lote inteiro, não a itens individuais):
- `400 Bad Request` — corpo malformado (ex.: `eventos` ausente, item sem `idLocal`).
- `401 Unauthorized` — token expirado ou inválido no momento da reconexão.

## `GET /api/v1/sync/status?idsLocais=<uuid1>,<uuid2>,...`

Consulta o estado de sincronização de um conjunto de `idLocal`, usado pela aplicação do motorista para confirmar, após reconexão, quais itens da fila local já podem ser removidos.

**Response 200 OK**:
```
[
  { "idLocal": "<uuid1>", "resultado": "ACEITO" },
  { "idLocal": "<uuid2>", "resultado": "NAO_ENCONTRADO" }
]
```

`NAO_ENCONTRADO` indica que o servidor nunca recebeu esse `idLocal` — a aplicação deve reenviá-lo via `POST /api/v1/sync/eventos`.
