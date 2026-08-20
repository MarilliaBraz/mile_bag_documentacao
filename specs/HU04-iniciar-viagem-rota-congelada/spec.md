# Feature Specification: HU04 — Iniciar a viagem com a rota congelada

**Feature Branch**: `HU04-iniciar-viagem-rota-congelada`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU de origem: `../../HUs/motorista/HU04-iniciar-viagem-rota-congelada.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF04), `../../requisitos/03-regras-negocio.md` (RN02), `../../requisitos/04-regras-dominio.md` (RD04). Spec guarda-chuva: `../001-milebag-user-stories/spec.md`. Depende de HU03 (`../HU03-calcular-e-escolher-rota/spec.md`) já concluída.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Iniciar a viagem com a rota congelada (Priority: P1)

Como motorista, eu quero iniciar a viagem com a rota escolhida, para que a quilometragem e os pedágios usados no meu pagamento fiquem garantidos, independentemente de qualquer consulta futura ao mapa.

**Why this priority**: É o requisito que resolve o problema central do projeto — a ausência de rastreabilidade do cálculo de pagamento. Sem o congelamento, o valor pago ao motorista fica sujeito a recomputações posteriores e à divergência que o sistema existe para eliminar.

**Independent Test**: Pode ser testada com uma rota já selecionada (HU03), iniciando a viagem e verificando que o snapshot gravado não muda mesmo que os dados de mapa mudem depois.

**Acceptance Scenarios**:

1. **Given** que uma rota foi selecionada para a entrega (HU03), **When** o motorista inicia a viagem, **Then** o sistema grava um snapshot com quilometragem de ida e volta, praças de pedágio e tempo estimado.
2. **Given** que a viagem já foi iniciada e o snapshot gravado, **When** o serviço de mapas ou a tabela de pedágios muda posteriormente, **Then** o snapshot da viagem permanece inalterado.
3. **Given** que uma viagem tem um snapshot gravado, **When** o valor a pagar por essa entrega é consultado, **Then** ele é sempre reconstituível a partir exclusivamente desse snapshot, nunca de uma nova consulta.

### Edge Cases

- O que acontece se o motorista tentar iniciar a viagem sem ter selecionado uma rota antes? O sistema deve impedir o início da viagem e indicar que uma rota precisa ser selecionada primeiro (HU03).
- O que acontece se o motorista tentar iniciar a viagem de uma entrega que já tem uma viagem iniciada? O sistema deve impedir um segundo início, mantendo o primeiro snapshot como o válido.
- O que acontece se não houver praça de pedágio cadastrada para um trecho identificado na rota? O snapshot deve ser gravado mesmo assim, sinalizando o trecho sem valor de pedágio cadastrado para tratamento posterior pelo administrador do tenant.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir iniciar a viagem somente a partir de uma entrega com rota já selecionada (HU03).
- **FR-002**: O sistema MUST gravar, ao iniciar a viagem, um snapshot com quilometragem de ida, quilometragem de volta, praças de pedágio identificadas e tempo estimado.
- **FR-003**: O sistema MUST tratar esse snapshot como imutável após a gravação — nenhuma operação subsequente pode alterá-lo.
- **FR-004**: O sistema MUST impedir o início de uma segunda viagem para uma entrega que já tenha uma viagem iniciada.
- **FR-005**: O sistema MUST permitir a reconstituição do valor a pagar por uma entrega exclusivamente a partir do snapshot gravado.
- **FR-006**: O sistema MUST sinalizar, no snapshot, qualquer trecho com pedágio identificado que não tenha valor cadastrado, sem impedir o início da viagem.

### Key Entities

- **Viagem (snapshot de rota)**: Registro imutável associado a uma Entrega, criado no início da viagem, contendo quilometragem de ida/volta, pedágios identificados e tempo estimado. Base exclusiva do cálculo de pagamento (ver `../001-milebag-user-stories/spec.md`, entidade "Viagem / snapshot de rota").

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das viagens iniciadas possuem um snapshot completo (quilometragem de ida/volta, pedágios, tempo estimado) gravado no momento do início.
- **SC-002**: Nenhuma alteração posterior em dados de mapa ou de pedágio modifica um snapshot já gravado, em nenhum caso observado.
- **SC-003**: O valor a pagar por qualquer entrega concluída pode ser recalculado, a qualquer momento, obtendo exatamente o mesmo resultado a partir do snapshot gravado.

## Assumptions

- O momento de "início da viagem" é uma ação explícita do motorista (não implícito ao selecionar a rota em HU03), permitindo que ele reveja a seleção antes do congelamento definitivo.
- O tratamento do trecho sem pedágio cadastrado (Edge Case) é resolvido operacionalmente pelo administrador do tenant após o fato, sem bloquear a operação do motorista em campo.
