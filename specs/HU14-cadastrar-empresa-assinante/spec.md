# Feature Specification: HU14 — Cadastrar empresa assinante (tenant)

**Feature Branch**: `HU14-cadastrar-empresa-assinante`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU-fonte `../../HUs/administrador-plataforma/HU14-cadastrar-empresa-assinante.md`

**Fontes**: `../../HUs/administrador-plataforma/HU14-cadastrar-empresa-assinante.md`, `../../requisitos/01-requisitos-funcionais.md` (RF10), `../../requisitos/03-regras-negocio.md` (RN09), `../../requisitos/02-requisitos-nao-funcionais.md` (RNF01, RNF10), `../../requisitos/04-regras-dominio.md` (RD01, RD11). Ver também [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), User Story 1, e as HUs relacionadas [`../HU15-convidar-primeiro-administrador-tenant/spec.md`](../HU15-convidar-primeiro-administrador-tenant/spec.md) (HU15) e [`../HU16-gerenciar-planos-assinatura/spec.md`](../HU16-gerenciar-planos-assinatura/spec.md) (HU16).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar uma nova empresa assinante (Priority: P1)

O administrador da plataforma cadastra uma nova empresa de entrega como tenant do sistema, associando-a a um plano de assinatura já existente. A partir desse cadastro, a empresa passa a existir como um espaço isolado de dados, pronto para receber seu primeiro administrador (HU15) e começar a operar — sem qualquer etapa de instalação técnica dedicada a ela.

**Why this priority**: É o ponto de entrada de qualquer empresa no sistema. Nenhuma outra funcionalidade do produto pode ser usada por uma empresa que ainda não foi cadastrada como tenant.

**Independent Test**: Pode ser testado cadastrando uma nova empresa com um plano associado e verificando que o registro do tenant existe, está isolado de outros tenants já cadastrados, e não deixa nenhuma pendência de configuração de infraestrutura em aberto.

**Acceptance Scenarios**:

1. **Given** que o administrador da plataforma está na tela de cadastro de empresas, **When** ele informa os dados da empresa e seleciona um plano de assinatura existente, **Then** o sistema cria o registro do tenant associado a esse plano.
2. **Given** que um novo tenant acabou de ser criado, **When** qualquer usuário de outro tenant já existente tenta acessar dados do novo tenant, **Then** o acesso é negado — o isolamento vale desde o primeiro registro.
3. **Given** que o cadastro foi concluído, **When** o administrador da plataforma verifica o status do novo tenant, **Then** nenhuma etapa de instalação, configuração de servidor ou intervenção técnica manual aparece como pendência para a empresa começar a operar.
4. **Given** que o administrador da plataforma tenta cadastrar uma empresa sem selecionar um plano de assinatura, **When** ele confirma o cadastro, **Then** o sistema recusa a operação e indica que um plano é obrigatório.

### Edge Cases

- O que acontece ao tentar cadastrar uma empresa com um identificador (ex.: CNPJ) já usado por outro tenant existente? O cadastro deve ser recusado, indicando o conflito, sem criar um segundo tenant para a mesma empresa.
- O que acontece se o plano de assinatura selecionado for removido ou desativado depois do cadastro do tenant? O tenant já cadastrado deve continuar operando normalmente; a alteração de plano é tratada por HU16, não por esta HU.
- O que acontece se o cadastro for interrompido antes de ser concluído (ex.: falha de rede do administrador da plataforma)? Nenhum tenant parcialmente configurado deve ficar visível ou operável — o cadastro é tudo-ou-nada.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador da plataforma cadastre uma nova empresa assinante, associando-a a um plano de assinatura existente.
- **FR-002**: O sistema MUST recusar o cadastro de um tenant sem um plano de assinatura associado.
- **FR-003**: O sistema MUST impedir o cadastro de dois tenants com o mesmo identificador de empresa.
- **FR-004**: O sistema MUST isolar os dados do novo tenant de todos os demais tenants desde o momento em que o registro é criado, sem período de transição.
- **FR-005**: O sistema MUST concluir o provisionamento de um novo tenant sem exigir qualquer etapa de instalação ou configuração de infraestrutura dedicada a essa empresa.
- **FR-006**: O sistema MUST tratar o cadastro do tenant como uma operação atômica — uma falha durante o cadastro não deve deixar um tenant parcialmente criado e visível.

### Key Entities

- **Tenant**: Empresa terceirizada de entrega, assinante do sistema. Identificador único de empresa, associado a exatamente um plano de assinatura vigente (RD11).
- **Plano de assinatura**: Ver [`../HU16-gerenciar-planos-assinatura/data-model.md`](../HU16-gerenciar-planos-assinatura/data-model.md) — referenciado, não redefinido aqui.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um administrador da plataforma consegue cadastrar uma nova empresa assinante sem qualquer etapa de instalação técnica, em uma única operação.
- **SC-002**: Em testes de isolamento realizados a cada novo tenant cadastrado, nenhum caso de acesso a dados de outro tenant é observado.
- **SC-003**: Tentativas de cadastro duplicado (mesmo identificador de empresa) ou sem plano associado são recusadas em 100% dos casos, sem criar registros parciais.

## Assumptions

- O identificador único de empresa usado para evitar duplicidade (FR-003) é um dado já coletado no cadastro (ex.: CNPJ), não definido nesta HU como um novo conceito.
- Esta HU não cobre o convite ao primeiro administrador do tenant nem a ativação do acesso — esse fluxo é tratado por HU15, disparado logo após o cadastro aqui descrito.
