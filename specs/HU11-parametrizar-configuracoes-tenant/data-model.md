# Fase 1 — Modelo de Dados: HU11

**Feature**: [spec.md](./spec.md)

## Entidade: MotivoOcorrencia

Catálogo por tenant referenciado por RD07 (`../../requisitos/04-regras-dominio.md`).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `descricao` | texto | sim | — |
| `ativo` | booleano | sim | `false` em vez de exclusão quando referenciado por ocorrências existentes (FR-009) |

## Entidade: CampoBaixaConfig

Um registro por campo disponível do formulário de baixa, indicando obrigatoriedade para o tenant (RI10, `../../requisitos/05-regras-interface.md`).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `campo` | enum (catálogo fixo do sistema) | sim | Ver Assumptions do spec.md — conjunto de campos é definido pelo sistema |
| `obrigatorio` | booleano | sim | Default conforme o campo (ex.: campos do POD já obrigatórios por RN04 do domínio) |

## Entidade: BaseAeroportuaria

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `codigoIata` | texto | sim | Código do aeroporto (ex.: `GYN`) |
| `nome` | texto | não | — |

## Entidade: CompanhiaAereaAtendida

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `codigoIata` | texto | sim | Código da companhia (ex.: `LA`, `G3`) |
| `nome` | texto | não | — |

## Entidade: PeriodicidadeFechamentoConfig

Histórico de vigência, mesmo padrão de [PracaPedagioValor](../HU10-cadastrar-pracas-pedagio/data-model.md).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | RNF01 |
| `periodicidade` | enum: `QUINZENAL` (default), outros valores futuros | sim | — |
| `vigenteDesde` | data | sim | O fechamento em andamento usa a periodicidade vigente no início do período (FR-008) |

**Relacionamentos**: todas as cinco entidades são N:1 com `Tenant`; `MotivoOcorrencia` é referenciada por `Ocorrência` (fora do escopo desta HU — ver domínio `ocorrencia` no [plano-mãe](../001-milebag-user-stories/plan.md)).
