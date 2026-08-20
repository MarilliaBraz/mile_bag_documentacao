# Fase 1 — Modelo de Dados: HU12

**Feature**: [spec.md](./spec.md)

## Entidade: Fechamento

Representa RD10 (`../../requisitos/04-regras-dominio.md`).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `periodoInicio` | data | sim | — |
| `periodoFim` | data | sim | — |
| `geradoEm` | data/hora | sim | — |
| `status` | enum: `COMPLETO`, `COM_PENDENCIAS` | sim | `COM_PENDENCIAS` quando ao menos um item está `pendente` (FR-006) |

**Regra de unicidade**: `(tenantId, periodoInicio, periodoFim)` é único (FR-005, research.md).

## Entidade: ItemFechamento

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `fechamentoId` | referência a Fechamento | sim | — |
| `entregaId` | referência a Entrega | sim | Entrega concluída com comprovante dentro do período (FR-004) |
| `motoristaId` | referência a Usuário | sim | — |
| `valorPagamentoEntrega` | decimal | condicional | Nulo quando `statusItem = PENDENTE` |
| `valorReembolsoPedagio` | decimal | condicional | Nulo quando `statusItem = PENDENTE`; sempre o dobro do valor de pedágio do snapshot (RN06) |
| `statusItem` | enum: `CALCULADO`, `PENDENTE` | sim | `PENDENTE` quando falta tarifa ou pedágio resolvível (FR-006) |
| `motivoPendencia` | texto | condicional | Preenchido quando `statusItem = PENDENTE` |

**Regras de validação**:
- `valorPagamentoEntrega` é derivado do snapshot de rota da entrega (quilometragem) e da tarifa resolvida (motorista > região > padrão — [HU09](../HU09-configurar-tabelas-tarifa/data-model.md)); nunca recalculado a partir de uma nova consulta (RN02).
- `valorReembolsoPedagio` é derivado dos valores de pedágio já congelados no snapshot da entrega (não uma nova consulta ao cadastro de [praças de pedágio](../HU10-cadastrar-pracas-pedagio/data-model.md) no momento do fechamento).

**Relacionamentos**:
- `Fechamento` 1:N `ItemFechamento`.
- `ItemFechamento` N:1 `Entrega` (entidade definida no domínio `entrega`, fora desta pasta — ver [spec-mãe](../001-milebag-user-stories/spec.md#key-entities)).
- `Fechamento` N:1 `Tenant`.

**Não modificado nesta HU**: `TabelaTarifa` ([HU09](../HU09-configurar-tabelas-tarifa/data-model.md)), `PracaPedagio`/`PracaPedagioValor` ([HU10](../HU10-cadastrar-pracas-pedagio/data-model.md)), `Entrega` e o snapshot de rota (spec-mãe) — todos apenas consumidos por leitura.

**Consumido por outra HU**: `Fechamento` é lido (nunca modificado) por [HU13](../HU13-emitir-relatorios-identidade-visual/data-model.md) no momento de renderizar o PDF do relatório, para aplicar a `IdentidadeVisualTenant` do tenant — não existe entidade própria de "relatório"; ver nota em `contracts/fechamento-api.md`.
