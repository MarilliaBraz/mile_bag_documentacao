# Fase 1 — Modelo de Dados

**Feature**: [spec.md](./spec.md)

## Entidade: Ocorrencia

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | gerado pelo sistema |
| `entregaId` | referência à Entrega | obrigatório |
| `motivoOcorrenciaId` | referência a MotivoOcorrencia | obrigatório; deve pertencer ao catálogo do mesmo tenant da entrega (FR-001) |
| `descricaoAdicional` | texto | opcional |
| `dataHora` | data/hora | obrigatório |
| `registradoOffline` | booleano | indica se o registro foi criado sem conexão (ver HU08) |

**Relacionamentos**:
- `Ocorrencia` N:1 `Entrega` — uma entrega pode acumular várias ocorrências ao longo de suas tentativas (FR-004).
- `Ocorrencia` N:1 `MotivoOcorrencia`.

**Regras de validação**:
- Rejeitar a criação se `entrega.status` já for `EntregaStatus.ENCERRADA_COM_SUCESSO` (FR-003).
- Ao ser criada com sucesso, a `Entrega` referenciada transita para `EntregaStatus.REABERTA` e o BDO correspondente volta ao estado que permite nova tentativa (FR-002, RD05 em `../../requisitos/04-regras-dominio.md`).
- Registros anteriores de `Ocorrencia` para a mesma entrega nunca são alterados ou removidos por uma nova ocorrência (FR-004).

**Nota de ciclo de vida**: RD05 descreve a reabertura como um retorno "ao início do ciclo", ou seja, `REABERTA` é funcionalmente equivalente a tornar a entrega disponível para uma nova autoatribuição (HU02) — inclusive por um motorista diferente do que fez a tentativa anterior. Isso exige que HU02 trate `REABERTA`, além de `AGUARDANDO_ATRIBUICAO`, como estado elegível para autoatribuir, e que `motoristaResponsavel` seja limpo na transição para `REABERTA` (ver nota equivalente em `../HU02-autoatribuir-entrega/data-model.md`) — sem isso, uma entrega reaberta fica presa, sem ninguém poder assumi-la de novo.

## Entidade: MotivoOcorrencia

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | gerado pelo sistema |
| `tenantId` | referência ao Tenant | obrigatório; escopa o catálogo (isolamento entre tenants) |
| `nome` | texto | obrigatório (ex.: "ausência do recebedor") |
| `ativo` | booleano | motivos inativos não aparecem para seleção em novas ocorrências |

Cadastro e edição de `MotivoOcorrencia` pertencem à HU11 (parametrização por tenant); esta HU apenas consome o catálogo já existente.

## Entidade: Entrega (referenciada, não redefinida aqui)

Ver o modelo completo, incluindo o enum canônico `EntregaStatus`, em `../HU02-autoatribuir-entrega/data-model.md`. Para esta HU, a Entrega expõe seu `status`, transitando para `EntregaStatus.REABERTA` via criação de uma `Ocorrencia`.
