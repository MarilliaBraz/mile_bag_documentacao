# Feature Specification: Vincular bagagem carona à entrega

**Feature Branch**: `HU05-vincular-bagagem-carona`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU05 — `../../HUs/motorista/HU05-vincular-bagagem-carona.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF05), `../../requisitos/03-regras-negocio.md` (RN03), `../../requisitos/04-regras-dominio.md` (RD03). Ver também a User Story 2 de [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), que já cobre este fluxo em conjunto com tarifas, pedágios e ocorrências.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Vincular bagagem carona à entrega em andamento (Priority: P1)

O motorista está a caminho de uma entrega já autoatribuída quando identifica que pode entregar, na mesma viagem, uma bagagem adicional de outro passageiro (bagagem carona). Ele vincula essa bagagem à entrega principal, sem precisar tratá-la como uma viagem separada.

**Why this priority**: É o único propósito desta HU — sem ela, o motorista é obrigado a abrir uma viagem completa nova para cada bagagem adicional, mesmo quando o deslocamento já está sendo feito.

**Independent Test**: Pode ser testado com uma entrega principal já autoatribuída e em andamento (HU02), capturando o BDO de uma segunda bagagem e vinculando-a a essa entrega, sem depender de rota calculada ou comprovante já registrado.

**Acceptance Scenarios**:

1. **Given** que uma entrega principal está autoatribuída e em andamento, **When** o motorista captura o BDO de uma bagagem adicional e escolhe vinculá-la a essa entrega, **Then** a bagagem passa a constar como carona da entrega principal.
2. **Given** que uma bagagem carona foi vinculada, **When** o fechamento do período for calculado, **Then** apenas a quilometragem complementar decorrente dessa bagagem é considerada, não uma viagem completa adicional.
3. **Given** que a entrega principal já foi encerrada com comprovante de entrega, **When** o motorista tenta vincular uma bagagem carona a ela, **Then** o sistema recusa a vinculação.

### Edge Cases

- O que acontece se o motorista tentar vincular a mesma bagagem carona a duas entregas principais diferentes? A segunda tentativa de vínculo deve ser recusada.
- O que acontece se a entrega principal for reaberta por ocorrência (HU07) depois de já ter uma bagagem carona vinculada? O vínculo deve ser preservado; a bagagem carona segue associada à mesma entrega principal após a reabertura.
- O que acontece se o motorista quiser desvincular uma bagagem carona por engano? Deve existir uma forma de remover o vínculo enquanto a entrega principal ainda não foi encerrada com comprovante.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o motorista vincule uma ou mais bagagens carona a uma entrega principal que esteja em andamento (autoatribuída e ainda não encerrada com comprovante).
- **FR-002**: O sistema MUST impedir a vinculação de uma bagagem carona a uma entrega principal já encerrada com comprovante de entrega.
- **FR-003**: O sistema MUST impedir que a mesma bagagem carona seja vinculada a mais de uma entrega principal simultaneamente.
- **FR-004**: O sistema MUST calcular, para cada bagagem carona, apenas a quilometragem complementar decorrente dela, distinta da quilometragem da entrega principal.
- **FR-005**: O sistema MUST preservar o vínculo entre bagagem carona e entrega principal mesmo quando a entrega principal for reaberta por ocorrência.

### Key Entities

- **Bagagem carona**: Bagagem adicional, originada de um BDO próprio, vinculada a uma entrega principal. Existe apenas em função dessa entrega — não é remunerada como entrega independente.
- **Entrega (principal)**: Entrega já existente (HU01–HU04) à qual uma ou mais bagagens carona podem ser vinculadas, enquanto permanecer em andamento.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um motorista consegue vincular uma bagagem carona a uma entrega em andamento em menos tempo do que levaria para abrir e conduzir uma entrega completa separada.
- **SC-002**: 100% das tentativas de vincular bagagem carona a uma entrega já encerrada com comprovante são recusadas pelo sistema.
- **SC-003**: No fechamento do período, o valor pago por uma bagagem carona reflete apenas a quilometragem complementar registrada, nunca o valor de uma viagem completa.

## Assumptions

- A quilometragem complementar de uma bagagem carona é definida no momento da vinculação (ex.: distância adicional até o endereço da bagagem carona), não recalculada após o fato.
- Uma entrega principal pode ter mais de uma bagagem carona vinculada; não há limite máximo definido pelas evidências do projeto.
