# Feature Specification: HU11 — Parametrizar configurações do tenant

**Feature Branch**: `HU11-parametrizar-configuracoes-tenant`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU fonte: `../../HUs/administrador-tenant/HU11-parametrizar-configuracoes-tenant.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF13), `../../requisitos/02-requisitos-nao-funcionais.md` (RNF01), `../../requisitos/04-regras-dominio.md` (RD07), `../../requisitos/05-regras-interface.md` (RI10). Contexto geral do produto: [spec-mãe](../001-milebag-user-stories/spec.md), User Story 3.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Parametrizar configurações operacionais do tenant (Priority: P1)

Como administrador do tenant, quero configurar os motivos de ocorrência, os campos obrigatórios do formulário de baixa, as bases aeroportuárias, as companhias aéreas atendidas e a periodicidade do fechamento, para que o sistema se adapte à forma como minha empresa opera, sem depender do fornecedor do sistema.

**Why this priority**: Única HU desta pasta. Corresponde à User Story 3 (P3) do spec-mãe, tratada aqui como P1 por ser a única fatia de valor desta especificação isolada.

**Independent Test**: Pode ser testado alterando cada uma das cinco configurações para um tenant já operacional e verificando que (a) a alteração é refletida onde a configuração é consumida (ex.: catálogo de ocorrências no app do motorista) e (b) nenhum outro tenant é afetado.

**Acceptance Scenarios**:

1. **Given** que o tenant não tem motivos de ocorrência cadastrados, **When** o administrador cadastra um catálogo de motivos, **Then** esse catálogo passa a ser o único exibido ao motorista ao registrar uma ocorrência para esse tenant (RD07).
2. **Given** que o tenant tem um formulário de baixa com campos padrão, **When** o administrador marca campos adicionais como obrigatórios, **Then** a interface de baixa exibida ao motorista passa a exigir esses campos (RI10).
3. **Given** que o tenant opera a partir de um aeroporto, **When** o administrador cadastra uma nova base aeroportuária, **Then** essa base passa a estar disponível para associação a entregas desse tenant.
4. **Given** que o tenant atende a uma companhia aérea, **When** o administrador cadastra essa companhia na lista de atendidas, **Then** BDOs dessa companhia passam a poder ser processados normalmente pelo tenant.
5. **Given** que o tenant usa a periodicidade quinzenal padrão, **When** o administrador altera a periodicidade do fechamento, **Then** os próximos fechamentos passam a seguir a nova periodicidade, sem afetar fechamentos já emitidos.
6. **Given** que um segundo tenant configura essas mesmas cinco categorias de forma diferente, **When** qualquer uma das duas empresas consulta suas configurações, **Then** cada uma vê exclusivamente as suas próprias (RNF01).

### Edge Cases

- O que acontece quando o administrador tenta remover um motivo de ocorrência que já foi usado em ocorrências registradas? O motivo deve ser desativado (deixa de aparecer para novos registros), não excluído, para preservar a integridade das ocorrências já registradas que o referenciam.
- O que acontece quando o administrador altera a periodicidade do fechamento no meio de um período em aberto? O período em aberto é concluído com a periodicidade anterior; a nova periodicidade vale a partir do próximo período (evita fechamentos parciais ou sobrepostos).
- O que acontece quando dois tenants cadastram a mesma base aeroportuária ou a mesma companhia aérea? Já coberto no spec-mãe (Edge Cases) — cada tenant mantém seu próprio cadastro, sem interferência.
- O que acontece quando o formulário de baixa exige um campo que o app do motorista, em uma versão desatualizada, não sabe exibir? Fora do escopo desta HU — tratado como requisito de compatibilidade entre versões do app, não de configuração.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador do tenant cadastre e edite um catálogo próprio de motivos de ocorrência.
- **FR-002**: O sistema MUST exibir ao motorista, ao registrar uma ocorrência, exclusivamente o catálogo de motivos configurado pelo tenant desse motorista.
- **FR-003**: O sistema MUST permitir que o administrador do tenant marque campos do formulário de baixa como obrigatórios ou opcionais.
- **FR-004**: O sistema MUST refletir, na interface de baixa exibida ao motorista, os campos obrigatórios configurados pelo tenant.
- **FR-005**: O sistema MUST permitir que o administrador do tenant cadastre e edite suas bases aeroportuárias.
- **FR-006**: O sistema MUST permitir que o administrador do tenant cadastre e edite a lista de companhias aéreas atendidas.
- **FR-007**: O sistema MUST permitir que o administrador do tenant configure a periodicidade do fechamento financeiro, com um valor padrão quinzenal.
- **FR-008**: O sistema MUST aplicar uma mudança de periodicidade apenas a partir do próximo período de fechamento, sem alterar períodos já em andamento ou já emitidos.
- **FR-009**: O sistema MUST desativar (não excluir) um motivo de ocorrência já referenciado por ocorrências existentes, quando removido pelo administrador.
- **FR-010**: O sistema MUST garantir que nenhuma das cinco configurações (motivos de ocorrência, campos obrigatórios, bases aeroportuárias, companhias atendidas, periodicidade) seja visível ou editável por um tenant diferente daquele que a configurou.

### Key Entities

- **Motivo de ocorrência**: catálogo por tenant, com status ativo/inativo (ver [data-model.md](./data-model.md)).
- **Configuração do formulário de baixa**: conjunto de campos e sua obrigatoriedade, por tenant.
- **Base aeroportuária**: cadastro por tenant.
- **Companhia aérea atendida**: cadastro por tenant.
- **Configuração de periodicidade de fechamento**: valor por tenant, com histórico de vigência.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador do tenant consegue alterar qualquer uma das cinco categorias de configuração sem apoio técnico externo.
- **SC-002**: Uma alteração de motivos de ocorrência ou de campos obrigatórios é refletida na interface do motorista na próxima vez que ele abre o formulário correspondente, sem necessidade de atualização manual do aplicativo.
- **SC-003**: Em teste de isolamento, nenhuma das cinco configurações de um tenant aparece nas telas de configuração de outro tenant.
- **SC-004**: Um fechamento em andamento no momento de uma mudança de periodicidade é concluído com as regras vigentes no início do período, sem exceção.

## Assumptions

- Cada categoria de configuração (motivos, campos, bases, companhias, periodicidade) tem um único conjunto de valores vigente por tenant — não há suporte a múltiplas configurações simultâneas dentro do mesmo tenant (ex.: por filial).
- O conjunto de campos disponíveis para marcar como obrigatórios no formulário de baixa é fixo, definido pelo sistema; o tenant escolhe quais, dentre esses, são obrigatórios — não cria campos novos livremente.
