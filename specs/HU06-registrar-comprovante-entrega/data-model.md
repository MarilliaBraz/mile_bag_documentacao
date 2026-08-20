# Fase 1 — Modelo de Dados

**Feature**: [spec.md](./spec.md)

## Entidade: ComprovanteEntrega (POD)

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | gerado pelo sistema |
| `entregaId` | referência à Entrega | obrigatório; único por entrega (FR-004) |
| `fotoDocumentoUrl` | referência à imagem armazenada | obrigatório (FR-001) |
| `identificacaoRecebedor` | texto | obrigatório (FR-001) |
| `dataHora` | data/hora | obrigatório; momento da captura, não do envio ao servidor |
| `latitude` / `longitude` | decimal | obrigatórios (FR-001); ver `research.md` para o tratamento de falha de captura |
| `registradoOffline` | booleano | indica se o registro foi criado sem conexão (ver HU08) |

**Relacionamentos**:
- `ComprovanteEntrega` 1:1 `Entrega` — uma entrega tem no máximo um comprovante de entrega válido.

**Regras de validação**:
- Rejeitar a criação se já existir um `ComprovanteEntrega` para a mesma `entregaId` (FR-004).
- Rejeitar a criação se qualquer campo obrigatório estiver ausente (FR-002).
- Ao ser criado com sucesso, a `Entrega` referenciada transita para `EntregaStatus.ENCERRADA_COM_SUCESSO` (FR-003, RD05 em `../../requisitos/04-regras-dominio.md`).

## Entidade: Entrega (referenciada, não redefinida aqui)

Ver o modelo completo, incluindo o enum canônico `EntregaStatus`, em `../HU02-autoatribuir-entrega/data-model.md`. Para esta HU, a Entrega expõe seu `status`, transitando para `EntregaStatus.ENCERRADA_COM_SUCESSO` apenas via criação de um `ComprovanteEntrega` válido.
