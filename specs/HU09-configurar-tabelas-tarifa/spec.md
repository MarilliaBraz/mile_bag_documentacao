# Feature Specification: HU09 — Configurar tabelas de tarifa

**Feature Branch**: `HU09-configurar-tabelas-tarifa`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU fonte: `../../HUs/administrador-tenant/HU09-configurar-tabelas-tarifa.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF08), `../../requisitos/03-regras-negocio.md` (RN07), `../../requisitos/04-regras-dominio.md` (RD09), `../../requisitos/02-requisitos-nao-funcionais.md` (RNF01). Contexto geral do produto: [spec-mãe](../001-milebag-user-stories/spec.md), User Story 2.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Configurar tabelas de tarifa do tenant (Priority: P1)

Como administrador do tenant, quero configurar as tarifas pagas por entrega, para que o valor pago a cada motorista reflita os acordos comerciais da minha empresa.

**Why this priority**: É a única HU desta pasta — corresponde à User Story 2 (P2) do spec-mãe, mas aqui é tratada isoladamente como P1 porque, nesta especificação, é a única fatia de valor em jogo. Sem tabela de tarifa não há como calcular o fechamento (HU12).

**Independent Test**: Pode ser testado cadastrando uma tarifa padrão do tenant e, em seguida, tarifas específicas por motorista e por região, verificando que cada uma se sobrepõe corretamente à padrão quando aplicável, sem depender do fechamento (HU12) estar implementado.

**Acceptance Scenarios**:

1. **Given** que o tenant ainda não tem tarifa padrão, **When** o administrador cadastra uma tarifa com valor fixo e valor por quilômetro, **Then** essa tarifa passa a ser a tarifa padrão do tenant.
2. **Given** que o tenant já tem uma tarifa padrão, **When** o administrador cadastra uma tarifa específica para um motorista, **Then** essa tarifa específica passa a prevalecer sobre a padrão para esse motorista.
3. **Given** que o tenant já tem uma tarifa padrão, **When** o administrador cadastra uma tarifa específica para uma região, **Then** essa tarifa passa a prevalecer sobre a padrão para entregas nessa região.
4. **Given** que um outro tenant também configurou suas próprias tarifas, **When** o administrador consulta as tarifas do seu tenant, **Then** somente as tarifas do seu próprio tenant são exibidas (RNF01).

### Edge Cases

- O que acontece quando uma entrega se enquadra simultaneamente em uma tarifa por motorista e em uma tarifa por região? A tarifa por motorista prevalece sobre a tarifa por região, que prevalece sobre a tarifa padrão do tenant — ordem de especificidade decrescente.
- O que acontece quando o administrador tenta cadastrar uma tarifa sem valor fixo nem valor por quilômetro? O cadastro deve ser rejeitado — ao menos um dos dois componentes é obrigatório.
- O que acontece quando uma tarifa é alterada depois que viagens já foram concluídas com a tarifa anterior? As viagens já concluídas não são recalculadas — apenas fechamentos futuros usam o novo valor (ver [snapshot de rota](../001-milebag-user-stories/spec.md), FR-007, que congela a base de cálculo de quilometragem; a tarifa aplicada no fechamento é a vigente no momento da apuração, não do início da viagem).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador do tenant cadastre uma tarifa padrão do tenant, com valor fixo, valor por quilômetro, ou ambos.
- **FR-002**: O sistema MUST permitir cadastrar tarifas específicas por motorista, que prevalecem sobre a tarifa padrão do tenant para esse motorista.
- **FR-003**: O sistema MUST permitir cadastrar tarifas específicas por região, que prevalecem sobre a tarifa padrão do tenant para entregas nessa região.
- **FR-004**: O sistema MUST resolver conflitos entre tarifas aplicáveis a uma mesma entrega pela ordem de especificidade: motorista > região > padrão do tenant.
- **FR-005**: O sistema MUST rejeitar o cadastro de uma tarifa sem valor fixo nem valor por quilômetro definido.
- **FR-006**: O sistema MUST garantir que as tarifas cadastradas por um tenant não sejam visíveis nem editáveis por outro tenant.

### Key Entities

- **Tabela de tarifa**: valores fixo e/ou por quilômetro, associada a um tenant e, opcionalmente, a um motorista específico ou a uma região específica (ver [data-model.md](./data-model.md)).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador do tenant consegue cadastrar uma tarifa completa (padrão, por motorista ou por região) sem apoio técnico externo, em uma única sessão de configuração.
- **SC-002**: Em uma entrega de teste sujeita a mais de uma tarifa aplicável, o sistema sempre resolve para a tarifa mais específica, sem intervenção manual.
- **SC-003**: Nenhuma tarifa de um tenant é visível a partir de uma sessão autenticada em outro tenant, em teste de isolamento.

## Assumptions

- Cada tenant tem no máximo uma tarifa padrão vigente por vez; múltiplas tarifas padrão simultâneas estão fora do escopo desta HU.
- "Região" é um valor de texto/categoria livre configurado pelo próprio tenant, sem uma tabela geográfica normalizada — está fora do escopo desta HU definir uma taxonomia de regiões.
