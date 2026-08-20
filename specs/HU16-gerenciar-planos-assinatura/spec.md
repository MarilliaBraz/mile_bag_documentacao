# Feature Specification: HU16 — Gerenciar planos de assinatura

**Feature Branch**: `HU16-gerenciar-planos-assinatura`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU-fonte `../../HUs/administrador-plataforma/HU16-gerenciar-planos-assinatura.md`

**Fontes**: `../../HUs/administrador-plataforma/HU16-gerenciar-planos-assinatura.md`, `../../requisitos/01-requisitos-funcionais.md` (RF12), `../../requisitos/03-regras-negocio.md` (RN10), `../../requisitos/04-regras-dominio.md` (RD11). Ver também [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), User Story 2, e a HU dependente [`../HU14-cadastrar-empresa-assinante/spec.md`](../HU14-cadastrar-empresa-assinante/spec.md) (HU14), que consome os planos definidos aqui.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar, associar e alterar planos de assinatura (Priority: P1)

O administrador da plataforma cadastra planos de assinatura, cada um com um limite de motoristas ativos e de volume mensal de BDOs processáveis. Cada tenant fica associado a exatamente um plano vigente, podendo ter esse plano alterado ao longo do tempo, sem que isso exija reimplantação do sistema.

**Why this priority**: Sem planos cadastrados, não é possível concluir o cadastro de um tenant (HU14 exige um plano) nem comercializar o serviço de forma diferenciada por porte de operação (RN10).

**Independent Test**: Pode ser testado cadastrando um plano com limites definidos, associando-o a um tenant existente, e depois trocando esse tenant para um segundo plano com limites diferentes — sem qualquer etapa de reimplantação.

**Acceptance Scenarios**:

1. **Given** que o administrador da plataforma está cadastrando um novo plano, **When** ele define o limite de motoristas ativos e o volume mensal de BDOs, **Then** o plano passa a existir e pode ser associado a tenants.
2. **Given** que um tenant já está associado a um plano vigente, **When** o administrador da plataforma altera esse tenant para um plano diferente, **Then** o novo plano passa a valer imediatamente, sem necessidade de reimplantação do sistema.
3. **Given** que um plano está associado a um ou mais tenants, **When** o administrador da plataforma tenta removê-lo, **Then** o sistema recusa a remoção enquanto houver tenant associado, orientando a migrar os tenants para outro plano primeiro.
4. **Given** que um tenant está associado a exatamente um plano, **When** qualquer consulta é feita sobre o plano vigente desse tenant, **Then** o sistema nunca retorna mais de um plano vigente simultâneo para o mesmo tenant.

### Edge Cases

- O que acontece quando um tenant já excedeu, no uso atual, o limite de motoristas ativos ou de volume de BDOs de um novo plano ao qual está sendo migrado? A migração deve ser permitida (a operação comercial pode decidir tolerar temporariamente o excesso), mas o sistema deve sinalizar o excedente ao administrador da plataforma e ao administrador do tenant, remetendo à aplicação do limite descrita em HU14/spec-mãe (FR-017).
- O que acontece se dois planos forem cadastrados com o mesmo nome? O sistema deve permitir (o nome não é identificador único), mas cada plano mantém um identificador interno distinto.
- O que acontece se o administrador da plataforma tentar definir um limite de motoristas ou de BDOs igual a zero ou negativo? O sistema deve recusar o valor, exigindo um limite positivo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador da plataforma cadastre um plano de assinatura com limite de motoristas ativos e limite de volume mensal de BDOs, ambos valores positivos.
- **FR-002**: O sistema MUST permitir associar um tenant a um plano de assinatura no momento do seu cadastro (consumido por HU14) e alterar essa associação posteriormente.
- **FR-003**: O sistema MUST garantir que cada tenant tenha, a qualquer momento, exatamente um plano vigente.
- **FR-004**: O sistema MUST aplicar a troca de plano de um tenant imediatamente, sem exigir reimplantação ou intervenção técnica.
- **FR-005**: O sistema MUST impedir a remoção de um plano enquanto houver ao menos um tenant associado a ele.
- **FR-006**: O sistema MUST sinalizar, ao administrador da plataforma e ao administrador do tenant, quando o uso atual de um tenant exceder os limites do plano recém-associado.

### Key Entities

- **Plano de assinatura**: Limite de motoristas ativos e de volume mensal de BDOs (RD11). Pode estar associado a nenhum, um ou vários tenants.
- **Tenant**: Ver [`../HU14-cadastrar-empresa-assinante/data-model.md`](../HU14-cadastrar-empresa-assinante/data-model.md) — cada tenant referencia exatamente um plano vigente.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um administrador da plataforma consegue cadastrar um plano e associá-lo a um tenant em poucos passos, sem qualquer etapa técnica de reimplantação.
- **SC-002**: 100% das tentativas de remoção de um plano com tenants associados são recusadas.
- **SC-003**: Em nenhum momento um tenant fica sem plano vigente ou com mais de um plano vigente simultâneo.

## Assumptions

- O ajuste automático (bloqueio de novas entregas, novos motoristas etc.) quando um tenant excede o limite de um plano é tratado como responsabilidade das HUs que criam esses recursos (ex.: cadastro de motorista, captura de BDO), não desta HU — aqui apenas a sinalização do excedente é especificada (FR-006).
- Não há suporte, nesta HU, a descontos, cobrança ou faturamento associado ao plano — apenas os limites operacionais (motoristas ativos, volume de BDOs), conforme RD11.
