# Fase 1 — Modelo de Dados

**Feature**: [spec.md](./spec.md)

## Entidade: RegistroPendente (fila local, no dispositivo do motorista — IndexedDB)

Não é uma entidade do backend; vive apenas no dispositivo até ser confirmada pelo servidor.

| Campo | Tipo | Regras |
|---|---|---|
| `idLocal` | UUID gerado no dispositivo | obrigatório; chave de idempotência (FR-006) — ver `research.md`, seção 1 |
| `sequenciaLocal` | inteiro, monotônico por dispositivo | obrigatório; define a ordem de processamento (FR-005) — ver `research.md`, seção 2 |
| `tipoEvento` | enumerado | um de: `BDO_CAPTURADO`, `ENTREGA_AUTOATRIBUIDA`, `ROTA_ESCOLHIDA`, `VIAGEM_INICIADA`, `BAGAGEM_CARONA_VINCULADA`, `COMPROVANTE_REGISTRADO`, `OCORRENCIA_REGISTRADA` |
| `payload` | objeto | dados específicos do domínio, no mesmo formato do corpo do endpoint síncrono correspondente (HU01–HU07) |
| `timestampCriacaoLocal` | data/hora | capturado no dispositivo no momento da ação |
| `status` | enumerado | `PENDENTE`, `SINCRONIZADO`, `ERRO` |
| `tentativas` | inteiro | incrementado a cada tentativa de envio malsucedida |

**Regras de validação**:
- Um `RegistroPendente` só é removido da fila local após receber confirmação `aceito` ou `duplicado` do servidor (FR-007).
- Itens com `status = ERRO` permanecem na fila para nova tentativa; a aplicação MUST retentar automaticamente quando a conectividade for detectada novamente.

## Entidade: EventoSincronizado (registro de processamento no servidor)

| Campo | Tipo | Regras |
|---|---|---|
| `idLocal` | UUID recebido do dispositivo | chave de deduplicação (único por tenant/dispositivo/motorista) |
| `tipoEvento` | enumerado | mesmo domínio de `RegistroPendente.tipoEvento` |
| `processadoEm` | data/hora | momento em que o servidor processou o evento |
| `resultado` | enumerado | `ACEITO`, `DUPLICADO`, `ERRO` |

Usado apenas para deduplicação e auditoria da sincronização; não substitui os registros de domínio (Entrega, ComprovanteEntrega, Ocorrência etc.), que continuam sendo a fonte de verdade de cada informação.
