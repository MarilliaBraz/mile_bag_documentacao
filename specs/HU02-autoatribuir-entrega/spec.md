# Feature Specification: HU02 — Autoatribuir a entrega

**Feature Branch**: `HU02-autoatribuir-entrega`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU de origem: `../../HUs/motorista/HU02-autoatribuir-entrega.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF02), `../../requisitos/04-regras-dominio.md` (RD05). Spec guarda-chuva: `../001-milebag-user-stories/spec.md`. Depende de HU01 (`../HU01-capturar-bdo-por-fotografia/spec.md`) já concluída.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Autoatribuir a entrega (Priority: P1)

Como motorista, eu quero atribuir a mim mesmo a entrega no momento em que capturo o BDO, para que eu assuma a responsabilidade pela entrega sem depender de distribuição manual pelo back-office.

**Why this priority**: Sem autoatribuição, um BDO capturado fica órfão — nenhum motorista é responsável por ele e o fluxo de rota/viagem/comprovante não tem a quem atribuir. É o elo entre a captura (HU01) e a execução da entrega (HU03 em diante).

**Independent Test**: Pode ser testada capturando um BDO (HU01) e verificando que, imediatamente após a confirmação, a opção de autoatribuição fica disponível e move a entrega para a lista do motorista.

**Acceptance Scenarios**:

1. **Given** que um BDO acabou de ser confirmado na tela de conferência (HU01), **When** o sistema apresenta a opção de autoatribuição, **Then** ela está disponível para o motorista sem etapas intermediárias.
2. **Given** que o motorista optou por autoatribuir a entrega, **When** a ação é confirmada, **Then** a entrega passa a constar na lista de entregas em andamento desse motorista.
3. **Given** que uma entrega já foi autoatribuída, **When** outro motorista tenta autoatribuí-la, **Then** o sistema impede a ação, pois a entrega já tem um responsável.

### Edge Cases

- O que acontece se o motorista fechar o aplicativo antes de confirmar a autoatribuição? O BDO confirmado permanece disponível para autoatribuição na próxima abertura do aplicativo, sem exigir nova captura.
- O que acontece se dois motoristas tentarem autoatribuir a mesma entrega quase simultaneamente (ex.: em cenário de reabertura de BDO)? Apenas o primeiro deve ter sucesso; o segundo recebe indicação de que a entrega já foi assumida.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST oferecer, imediatamente após a confirmação de um BDO, a opção de o motorista atribuir a si mesmo a entrega correspondente.
- **FR-002**: O sistema MUST registrar o motorista responsável ao confirmar a autoatribuição.
- **FR-003**: O sistema MUST listar, para cada motorista, as entregas que ele autoatribuiu e que estão em andamento.
- **FR-004**: O sistema MUST impedir que uma entrega já atribuída a um motorista seja autoatribuída por outro.

### Key Entities

- **Entrega**: Liga um BDO confirmado a um motorista responsável. Atributo relevante nesta HU: `motoristaResponsavel` (nulo até a autoatribuição).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Todo BDO confirmado tem, no máximo, um motorista responsável a qualquer momento.
- **SC-002**: O motorista consegue autoatribuir uma entrega em uma única ação, sem navegação adicional além da tela de conferência do BDO.

## Assumptions

- A autoatribuição é a única forma de atribuição de entrega prevista nesta fase — não há distribuição manual pelo back-office no escopo desta HU.
