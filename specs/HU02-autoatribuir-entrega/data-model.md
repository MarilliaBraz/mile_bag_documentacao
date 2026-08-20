# Fase 1 — Modelo de Dados: HU02

**Feature**: [spec.md](./spec.md)

## Entidade: Entrega

Liga um BDO confirmado (ver `../HU01-capturar-bdo-por-fotografia/data-model.md`) a um motorista responsável. Ver `../../requisitos/04-regras-dominio.md` (RD05, ciclo de vida da entrega).

| Campo | Descrição | Regra |
|---|---|---|
| id | Identificador único da entrega | — |
| tenantId | Tenant proprietário | Obrigatório; isolamento RNF01 |
| bdoId | Referência ao BDO de origem | Obrigatório, 1:1 com o BDO confirmado |
| motoristaResponsavel | Motorista que autoatribuiu a entrega | Nulo até a autoatribuição; imutável dentro do escopo desta HU (sem reatribuição direta) — a única exceção é a reabertura por ocorrência (HU07), que limpa este campo como parte da transição para `REABERTA` |
| status | Estado da entrega no ciclo de vida (ver RD05) — ver enum `EntregaStatus` abaixo | Nesta HU, transita de `AGUARDANDO_ATRIBUICAO` **ou** `REABERTA` para `ATRIBUIDA` |

**Validações**:
- `motoristaResponsavel` só pode ser definido a partir de um estado nulo; tentativa de nova atribuição quando já preenchido MUST ser rejeitada (FR-004).
- A definição de `motoristaResponsavel` é sempre o usuário autenticado da requisição, nunca um valor recebido explicitamente no corpo da requisição.
- Autoatribuição é permitida quando `status` for `AGUARDANDO_ATRIBUICAO` OU `REABERTA` (RD05 — reabertura retorna a entrega "ao início do ciclo"; ver nota em `../HU07-registrar-ocorrencia-nao-entrega/data-model.md`). Isso permite que um motorista diferente do que fez a tentativa anterior assuma a nova tentativa.

### Enum `EntregaStatus` (canônico)

`Entrega` é definida pela primeira vez nesta HU (HU02), então este é o local canônico do enum de status do ciclo de vida completo da entrega — as demais HUs (HU04, HU06, HU07) referenciam estes mesmos valores por link relativo a este arquivo, em vez de redefini-los.

| Valor | Quando é produzido | HU responsável |
|---|---|---|
| `AGUARDANDO_ATRIBUICAO` | BDO confirmado (HU01), ainda sem motorista | HU01 |
| `ATRIBUIDA` | Motorista autoatribui a entrega | HU02 (esta HU) |
| `EM_ANDAMENTO` | Rota escolhida e viagem congelada (snapshot) | HU04 |
| `ENCERRADA_COM_SUCESSO` | Comprovante de entrega (POD) registrado | HU06 |
| `REABERTA` | Ocorrência de não entrega registrada | HU07 |

Convenção: valores em `SCREAMING_SNAKE_CASE`, por `mile_bag_audite/CONVENTIONS.md` §4.1 (constantes/enums Java).

**Relacionamentos**: uma Entrega referencia exatamente um BDO (HU01). Recebe, nas próximas HUs, uma Viagem (HU03/HU04) e um Comprovante ou Ocorrência (HU06/HU07).
