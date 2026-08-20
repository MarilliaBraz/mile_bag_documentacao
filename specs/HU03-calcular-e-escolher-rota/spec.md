# Feature Specification: HU03 — Calcular e escolher a rota da entrega

**Feature Branch**: `HU03-calcular-e-escolher-rota`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU de origem: `../../HUs/motorista/HU03-calcular-e-escolher-rota.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF03), `../../requisitos/05-regras-interface.md` (RI03). Spec guarda-chuva: `../001-milebag-user-stories/spec.md`. Depende de HU02 (`../HU02-autoatribuir-entrega/spec.md`) já concluída.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Calcular e escolher a rota da entrega (Priority: P1)

Como motorista, eu quero ver as rotas sugeridas até o endereço de entrega, com distância, tempo e pedágios, para que eu possa escolher a rota mais adequada ou traçar uma rota própria.

**Why this priority**: É pré-requisito direto do congelamento da rota (HU04): sem uma rota calculada e escolhida, não há o que congelar. Também é onde o motorista ganha autonomia operacional em campo.

**Independent Test**: Pode ser testada com uma entrega já autoatribuída (HU02), solicitando o cálculo de rota e validando que distância, tempo e pedágios aparecem corretamente para o endereço do BDO.

**Acceptance Scenarios**:

1. **Given** que uma entrega está autoatribuída ao motorista, **When** ele solicita o cálculo da rota até o endereço de entrega, **Then** o sistema apresenta ao menos uma rota sugerida com distância, tempo estimado e trechos com pedágio identificados.
2. **Given** que mais de uma rota alternativa está disponível, **When** o motorista revisa as opções, **Then** ele pode selecionar qualquer uma delas como a rota da entrega.
3. **Given** que nenhuma das rotas sugeridas atende à situação real do motorista, **When** ele opta por traçar uma rota própria, **Then** o sistema permite o traçado manual e solicita uma justificativa para o registro.

### Edge Cases

- O que acontece quando o endereço do BDO não pode ser geocodificado (endereço incompleto ou não localizado)? O motorista deve poder ajustar manualmente o ponto de entrega no mapa antes de calcular a rota.
- O que acontece quando não há cobertura de dados de mapa suficiente na região (área rural, por exemplo)? O motorista deve poder traçar rota própria mesmo sem rota sugerida disponível.
- O que acontece se o motorista solicitar o cálculo de rota mais de uma vez antes de iniciar a viagem? Cada novo cálculo substitui as opções anteriores; nada é congelado até o início da viagem (HU04).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST calcular, a partir do endereço de entrega do BDO, ao menos uma rota sugerida com distância, tempo estimado e trechos com pedágio.
- **FR-002**: O sistema MUST apresentar rotas alternativas quando existir mais de uma opção viável.
- **FR-003**: O sistema MUST permitir ao motorista selecionar uma das rotas sugeridas como a rota da entrega.
- **FR-004**: O sistema MUST permitir ao motorista traçar uma rota própria, diferente das sugeridas.
- **FR-005**: O sistema MUST solicitar uma justificativa do motorista ao optar por rota própria.
- **FR-006**: O sistema MUST permitir o ajuste manual do ponto de entrega no mapa quando o endereço não puder ser geocodificado automaticamente.

### Key Entities

- **Rota (proposta)**: Opção de trajeto calculada para uma entrega, com distância, tempo estimado e trechos com pedágio. Não é definitiva até a seleção do motorista — o registro definitivo (imutável) só existe a partir de HU04.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Para entregas com endereço geocodificável, ao menos uma rota sugerida é apresentada em praticamente todas as solicitações.
- **SC-002**: O motorista consegue concluir a escolha da rota (sugerida ou própria) em uma única tela, sem alternar entre múltiplas telas para comparar as opções.

## Assumptions

- O cálculo de rota depende de um serviço de mapas com cobertura da região de atuação do tenant; a qualidade da cobertura fora dessa região está fora do escopo de garantia desta HU.
- A rota própria traçada pelo motorista é aceita sem validação automática de plausibilidade nesta fase — a justificativa textual é o único controle previsto.
