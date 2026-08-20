# Feature Specification: HU10 — Cadastrar praças de pedágio

**Feature Branch**: `HU10-cadastrar-pracas-pedagio`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU fonte: `../../HUs/administrador-tenant/HU10-cadastrar-pracas-pedagio.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF11), `../../requisitos/03-regras-negocio.md` (RN02, RN06), `../../requisitos/04-regras-dominio.md` (RD08). Contexto geral do produto: [spec-mãe](../001-milebag-user-stories/spec.md), User Story 2.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar praças de pedágio do tenant (Priority: P1)

Como administrador do tenant, quero cadastrar as praças de pedágio e seus valores, para que o reembolso de pedágio pago aos motoristas seja calculado corretamente.

**Why this priority**: Única HU desta pasta. É pré-requisito para que o snapshot de rota (spec-mãe, FR-007) tenha valores de pedágio a atribuir, e para que o fechamento (HU12) calcule o reembolso de pedágio.

**Independent Test**: Pode ser testado cadastrando uma praça com valor e data de vigência, atualizando esse valor posteriormente, e conferindo que o histórico de valores anteriores permanece consultável — sem depender do cálculo de rota nem do fechamento estarem implementados.

**Acceptance Scenarios**:

1. **Given** que uma praça de pedágio ainda não está cadastrada, **When** o administrador cadastra a praça com um valor e uma data de vigência, **Then** a praça passa a existir no cadastro do tenant, disponível para ser referenciada em rotas futuras.
2. **Given** que uma praça já tem um valor vigente, **When** o administrador cadastra um novo valor com uma nova data de vigência, **Then** o valor anterior é preservado com sua data de vigência original, e não é sobrescrito.
3. **Given** que uma viagem já foi congelada com o valor de pedágio vigente na época, **When** o valor da praça é atualizado posteriormente, **Then** o snapshot da viagem já congelada permanece inalterado.
4. **Given** que o reembolso de uma viagem está sendo calculado, **When** o sistema aplica o valor da praça, **Then** o valor é sempre contabilizado em dobro (ida e volta), conforme RN06.

### Edge Cases

- O que acontece quando o administrador cadastra uma nova data de vigência anterior à data de vigência de um valor já existente para a mesma praça? O cadastro deve ser rejeitado ou exigir confirmação explícita, para não introduzir ambiguidade sobre qual valor vale em uma data específica.
- O que acontece quando uma rota passa por uma praça de pedágio ainda não cadastrada no tenant? Tratado no spec-mãe (Edge Cases) — a viagem pode ser congelada mesmo assim, sinalizando o trecho sem valor cadastrado.
- O que acontece quando duas praças de pedágio diferentes têm o mesmo nome em tenants diferentes? Cada tenant mantém seu próprio cadastro; não há colisão entre tenants (RNF01).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador do tenant cadastre uma praça de pedágio com nome/identificação, valor e data de vigência.
- **FR-002**: O sistema MUST preservar todo valor histórico de uma praça de pedágio, associado à sua data de vigência original, quando um novo valor é cadastrado.
- **FR-003**: O sistema MUST usar, no congelamento de uma viagem, o valor de pedágio vigente na data da viagem — nunca um valor cadastrado posteriormente.
- **FR-004**: O sistema MUST calcular o reembolso de pedágio de uma viagem sempre considerando o trajeto de ida e de volta (RN06).
- **FR-005**: O sistema MUST rejeitar o cadastro de um novo valor de vigência para uma praça quando a data de vigência informada for anterior à do valor vigente mais recente já cadastrado.
- **FR-006**: O sistema MUST garantir que as praças de pedágio cadastradas por um tenant não sejam visíveis nem editáveis por outro tenant.

### Key Entities

- **Praça de pedágio**: identificação, tenant, e um histórico de valores com data de vigência (ver [data-model.md](./data-model.md)).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador consegue cadastrar uma praça de pedágio e seu valor em uma única operação, sem apoio técnico externo.
- **SC-002**: Ao atualizar o valor de uma praça já usada em viagens passadas, 100% dos snapshots de viagens anteriores permanecem com o valor original (verificável por amostragem).
- **SC-003**: O valor de reembolso calculado para qualquer viagem sempre corresponde ao dobro do valor de pedágio vigente na data da viagem (ida + volta), sem exceção.

## Assumptions

- O valor de uma praça de pedágio é único por sentido (não há tarifas diferentes para ida e volta na mesma praça) — o dobro (RN06) é aplicado sobre esse único valor.
- A "região" ou rodovia à qual uma praça pertence não é modelada nesta HU; a praça é identificada apenas por nome/localização, cadastrado livremente pelo tenant.
