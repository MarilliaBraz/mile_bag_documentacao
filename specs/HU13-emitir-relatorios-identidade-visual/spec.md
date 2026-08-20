# Feature Specification: HU13 — Emitir relatórios com identidade visual do tenant

**Feature Branch**: `HU13-emitir-relatorios-identidade-visual`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU-fonte `../../HUs/administrador-tenant/HU13-emitir-relatorios-identidade-visual.md`

**Fontes**: `../../HUs/administrador-tenant/HU13-emitir-relatorios-identidade-visual.md`, `../../requisitos/01-requisitos-funcionais.md` (RF15), `../../requisitos/05-regras-interface.md` (RI07). Ver também a especificação consolidada em [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), User Story 3, e o fechamento quinzenal em [`../HU12-gerar-fechamento-quinzenal/spec.md`](../HU12-gerar-fechamento-quinzenal/spec.md) (HU12), do qual esta HU depende.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Emitir relatórios com a identidade visual do tenant (Priority: P1)

O administrador do tenant configura a identidade visual (logotipo/cabeçalho) da sua empresa. A partir dessa configuração, todo relatório de fechamento gerado para o tenant passa a sair automaticamente com essa identidade aplicada, sem que o administrador precise formatar ou ajustar o arquivo manualmente antes de enviá-lo à companhia aérea contratante.

**Why this priority**: É a única fatia de valor desta HU — sem ela, o relatório sai com aparência genérica e o administrador precisa retrabalhar a formatação a cada fechamento antes de enviá-lo ao cliente.

**Independent Test**: Pode ser testado configurando a identidade visual de um tenant já operacional (que já gera fechamentos, ver HU12) e verificando que o próximo relatório emitido reflete essa identidade, sem alterar o cálculo dos valores do fechamento.

**Acceptance Scenarios**:

1. **Given** que o administrador do tenant está nas configurações da empresa, **When** ele envia um logotipo e define os dados de cabeçalho, **Then** o sistema salva essa identidade visual associada exclusivamente ao seu tenant.
2. **Given** que o tenant já tem uma identidade visual configurada, **When** um relatório de fechamento é gerado para esse tenant, **Then** o relatório é emitido com o logotipo e o cabeçalho configurados, sem exigir seleção manual a cada emissão.
3. **Given** que dois tenants diferentes têm identidades visuais distintas configuradas, **When** cada um gera seu relatório de fechamento, **Then** cada relatório reflete apenas a identidade visual do próprio tenant, nunca a de outro.
4. **Given** que um tenant ainda não configurou identidade visual própria, **When** ele gera um relatório de fechamento, **Then** o relatório é emitido com uma aparência padrão neutra, sem erro e sem bloquear a emissão.

### Edge Cases

- O que acontece quando o administrador envia um arquivo de logotipo em formato ou tamanho não suportado? O sistema deve rejeitar o envio com uma mensagem clara, sem corromper a identidade visual já configurada anteriormente.
- O que acontece quando o administrador atualiza a identidade visual depois que um fechamento já foi gerado? O relatório já emitido não deve ser alterado retroativamente; apenas os relatórios gerados a partir da atualização usam a nova identidade.
- O que acontece se o tenant remover a identidade visual configurada? Os relatórios seguintes voltam a usar a aparência padrão neutra (cenário 4 acima).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o administrador do tenant configure uma identidade visual própria (logotipo e dados de cabeçalho), associada exclusivamente ao seu tenant.
- **FR-002**: O sistema MUST aplicar automaticamente a identidade visual configurada a todo relatório de fechamento gerado para esse tenant, sem exigir seleção manual a cada emissão.
- **FR-003**: O sistema MUST impedir que a identidade visual de um tenant seja aplicada ou fique visível nos relatórios de outro tenant.
- **FR-004**: O sistema MUST permitir a emissão de relatórios com uma aparência padrão neutra quando o tenant não tiver configurado identidade visual própria.
- **FR-005**: O sistema MUST preservar a identidade visual usada em relatórios já emitidos, mesmo que a configuração do tenant seja alterada posteriormente.

### Key Entities

- **Identidade visual do tenant**: Configuração (logotipo, dados de cabeçalho) associada a um tenant, usada na geração de relatórios de fechamento.
- **Relatório de fechamento**: Ver [`../HU12-gerar-fechamento-quinzenal/data-model.md`](../HU12-gerar-fechamento-quinzenal/data-model.md) — esta HU apenas adiciona a aparência aplicada na emissão, não altera seu conteúdo financeiro.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um administrador de tenant consegue configurar sua identidade visual e ver o próximo relatório de fechamento emitido com ela aplicada, sem etapas manuais de formatação.
- **SC-002**: Em uma amostra de relatórios de tenants distintos, 100% deles refletem apenas a identidade visual do próprio tenant.

## Assumptions

- A identidade visual é composta por, no mínimo, um logotipo e um cabeçalho textual (nome/dados da empresa); elementos adicionais (cores, rodapé) podem ser incorporados futuramente sem alterar esta especificação.
- Esta HU depende da existência do relatório de fechamento (HU12) como superfície onde a identidade visual é aplicada; não introduz um novo tipo de relatório.
