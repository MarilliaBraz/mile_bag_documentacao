# Feature Specification: HU15 — Convidar o primeiro administrador do tenant

**Feature Branch**: `HU15-convidar-primeiro-administrador-tenant`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU-fonte `../../HUs/administrador-plataforma/HU15-convidar-primeiro-administrador-tenant.md`

**Fontes**: `../../HUs/administrador-plataforma/HU15-convidar-primeiro-administrador-tenant.md`, `../../requisitos/01-requisitos-funcionais.md` (RF10), `../../requisitos/05-regras-interface.md` (RI09), `../../requisitos/04-regras-dominio.md` (RD12). Ver também [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), User Story 1, e a HU antecessora [`../HU14-cadastrar-empresa-assinante/spec.md`](../HU14-cadastrar-empresa-assinante/spec.md) (HU14), da qual esta HU depende diretamente.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Convidar automaticamente o primeiro administrador ao concluir o cadastro do tenant (Priority: P1)

Assim que o cadastro de um novo tenant é concluído (HU14), o sistema dispara automaticamente um convite ao primeiro administrador indicado para aquele tenant. O convite permite que essa pessoa defina suas próprias credenciais de acesso e assuma o papel de administrador do tenant, sem que o administrador da plataforma precise executar nenhuma etapa manual adicional.

**Why this priority**: Sem esta HU, um tenant recém-cadastrado (HU14) fica sem ninguém capaz de acessá-lo — o cadastro por si só não entrega valor de negócio até que alguém consiga operar o tenant.

**Independent Test**: Pode ser testado cadastrando um tenant com um e-mail de administrador indicado e verificando que o convite é enviado automaticamente, sem ação manual, e que o link do convite permite ativar o acesso.

**Acceptance Scenarios**:

1. **Given** que o cadastro de um novo tenant foi concluído com um administrador indicado, **When** o cadastro é finalizado, **Then** um convite é disparado automaticamente para o destinatário indicado, sem exigir uma ação separada do administrador da plataforma.
2. **Given** que o destinatário recebeu o convite, **When** ele acessa o link do convite e define suas credenciais de acesso, **Then** sua conta é criada com o perfil de administrador do tenant correspondente.
3. **Given** que um convite já foi usado para ativar uma conta, **When** alguém tenta acessar o mesmo link de convite novamente, **Then** o sistema recusa o uso repetido do convite.
4. **Given** que um convite não foi usado dentro do prazo de validade, **When** o destinatário tenta acessá-lo depois de expirado, **Then** o sistema informa que o convite expirou e orienta a solicitar um novo.

### Edge Cases

- O que acontece se o cadastro do tenant (HU14) informar um e-mail de administrador inválido ou malformado? O convite não deve ser enviado, e o cadastro do tenant deve sinalizar essa pendência para correção, sem deixar o tenant sem nenhum caminho de ativação.
- O que acontece se o destinatário perder o convite antes de usá-lo? O administrador da plataforma deve poder reenviar um novo convite para o mesmo tenant, invalidando o anterior.
- O que acontece se dois convites diferentes forem gerados para o mesmo tenant antes de qualquer um deles ser usado? Apenas o convite mais recente deve permanecer válido; o anterior é invalidado automaticamente.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST disparar automaticamente um convite ao primeiro administrador indicado, imediatamente após a conclusão do cadastro do tenant (HU14), sem exigir ação manual adicional.
- **FR-002**: O sistema MUST permitir que o destinatário do convite defina suas próprias credenciais de acesso ao ativá-lo.
- **FR-003**: O sistema MUST associar a conta criada a partir do convite ao perfil de administrador do tenant correspondente, e a nenhum outro tenant.
- **FR-004**: O sistema MUST invalidar um convite depois de usado, impedindo reutilização.
- **FR-005**: O sistema MUST expirar um convite não utilizado após um prazo definido, informando claramente ao destinatário quando isso ocorrer.
- **FR-006**: O sistema MUST permitir que o administrador da plataforma reenvie um convite para um tenant sem administrador ativo, invalidando qualquer convite anterior ainda pendente para esse tenant.

### Key Entities

- **Convite de administrador**: Registro de convite associado a um tenant e a um destinatário, com estado (pendente, usado, expirado, invalidado) e prazo de validade.
- **Usuário / Perfil**: Ver [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md#key-entities) — esta HU é o mecanismo que cria a primeira conta de administrador de um tenant.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos tenants cadastrados com sucesso (HU14) têm um convite de administrador disparado automaticamente, sem intervenção manual.
- **SC-002**: Um destinatário consegue ativar seu acesso e começar a operar o tenant em poucos minutos após receber o convite, sem depender de suporte técnico.
- **SC-003**: Nenhum convite é aceito mais de uma vez, e nenhum convite expirado é aceito.

## Assumptions

- O canal de entrega do convite é e-mail (ou mecanismo equivalente já assumido no spec-mãe); o formato exato da mensagem não é definido nesta HU.
- O prazo de validade do convite (FR-005) é um valor de configuração da plataforma, não uma constante fixa no código — a definição do valor exato fica para a fase de implementação.
- Esta HU cobre apenas o primeiro administrador de um tenant; convites para administradores adicionais do mesmo tenant (por um administrador já ativo) estão fora do escopo desta HU.
