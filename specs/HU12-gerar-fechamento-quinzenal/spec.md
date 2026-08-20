# Feature Specification: HU12 — Gerar o fechamento quinzenal

**Feature Branch**: `HU12-gerar-fechamento-quinzenal`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU fonte: `../../HUs/administrador-tenant/HU12-gerar-fechamento-quinzenal.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF09), `../../requisitos/03-regras-negocio.md` (RN02, RN06, RN07, RN08), `../../requisitos/04-regras-dominio.md` (RD04, RD09, RD10). Contexto geral do produto: [spec-mãe](../001-milebag-user-stories/spec.md), User Story 2. Depende diretamente de [HU09 — tarifas](../HU09-configurar-tabelas-tarifa/spec.md) e [HU10 — pedágios](../HU10-cadastrar-pracas-pedagio/spec.md).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Gerar o fechamento periódico de pagamento (Priority: P1)

Como administrador do tenant, quero gerar o fechamento periódico de pagamento de entregas e de reembolso de pedágios, para que eu saiba exatamente quanto devo pagar a cada motorista, sem apuração manual.

**Why this priority**: Única HU desta pasta — é o ponto em que os dados registrados durante a operação (entregas com snapshot congelado, tarifas, pedágios) se transformam em valor a pagar. É o motivo de ser do "módulo financeiro" (spec-mãe, User Story 2).

**Independent Test**: Pode ser testado com um tenant que já tenha entregas concluídas com snapshot de rota congelado (dependência de HU04 no spec-mãe) e tarifas/pedágios cadastrados (HU09, HU10) — gerando o fechamento do período e conferindo os totais por motorista contra um cálculo manual de referência.

**Acceptance Scenarios**:

1. **Given** que o período de fechamento se encerrou e existem entregas concluídas nesse período, **When** o administrador solicita a geração do fechamento, **Then** o sistema calcula, para cada motorista, o valor de pagamento de entregas e o valor de reembolso de pedágios, separadamente.
2. **Given** que uma entrega tem snapshot de rota congelado, **When** o fechamento calcula o valor dessa entrega, **Then** o cálculo usa exclusivamente os valores do snapshot (quilometragem, pedágios) — nunca uma nova consulta ao serviço de mapas (RN02).
3. **Given** que uma entrega está sujeita a mais de uma tarifa aplicável (motorista, região, padrão), **When** o fechamento calcula o valor de pagamento, **Then** aplica a tarifa resolvida com a mesma regra de especificidade de [HU09](../HU09-configurar-tabelas-tarifa/spec.md).
4. **Given** que o fechamento foi gerado, **When** o administrador consulta o resultado, **Then** os relatórios de pagamento de entregas e de reembolso de pedágios são apresentados como documentos distintos (RN08).
5. **Given** que um fechamento já foi gerado para um período, **When** o administrador tenta gerar novamente o fechamento do mesmo período, **Then** o sistema impede a duplicação ou reaproveita o fechamento já existente, nunca produzindo dois fechamentos para o mesmo período e tenant.
6. **Given** que uma entrega ainda está em andamento (sem comprovante nem ocorrência) ao final do período, **When** o fechamento é gerado, **Then** essa entrega é excluída do fechamento atual e considerada no próximo período em que for concluída.

### Edge Cases

- O que acontece quando uma bagagem carona (spec-mãe, FR-008) está vinculada a uma entrega concluída dentro do período? Seu valor complementar de quilometragem entra no cálculo de pagamento da entrega principal à qual está vinculada, não como uma linha separada no fechamento.
- O que acontece quando uma entrega foi reaberta por ocorrência mais de uma vez (spec-mãe, FR-011) e só foi concluída em um período posterior? Somente a conclusão (com comprovante) dentro de um período conta para o fechamento desse período; tentativas anteriores malsucedidas não geram pagamento.
- O que acontece quando não existe tarifa nem valor de pedágio cadastrado para alguma entrega do período? O fechamento deve sinalizar essa entrega como pendente de configuração, sem bloquear a geração do restante do fechamento nem atribuir um valor arbitrário.
- O que acontece quando duas entregas do mesmo motorista, no mesmo período, usam tarifas diferentes (uma por região, outra pela padrão)? Cada entrega usa a tarifa resolvida individualmente para ela (FR-004 de HU09); o fechamento soma os valores já calculados por entrega.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST calcular, para um período de fechamento, o valor de pagamento de entregas devido a cada motorista, com base nos snapshots de rota congelados e nas tarifas resolvidas para cada entrega concluída no período.
- **FR-002**: O sistema MUST calcular, para o mesmo período, o valor de reembolso de pedágio devido a cada motorista, com base nos valores de pedágio congelados no snapshot de cada entrega.
- **FR-003**: O sistema MUST apresentar o valor de pagamento de entregas e o valor de reembolso de pedágios como totais separados, nunca somados em uma única linha.
- **FR-004**: O sistema MUST considerar, no fechamento de um período, apenas entregas encerradas com comprovante dentro desse período.
- **FR-005**: O sistema MUST impedir a geração de mais de um fechamento para o mesmo tenant e o mesmo período.
- **FR-006**: O sistema MUST sinalizar, sem bloquear a geração do restante do fechamento, qualquer entrega do período para a qual não exista tarifa ou valor de pedágio aplicável.
- **FR-007**: O sistema MUST gerar o fechamento sem qualquer intervenção manual de ajuste de valores por parte do administrador.
- **FR-008**: O sistema MUST garantir que o fechamento de um tenant nunca inclua entregas, tarifas ou pedágios de outro tenant.

### Key Entities

- **Fechamento**: agregação periódica de pagamento de entregas e reembolso de pedágios por motorista, calculada e emitida uma única vez por tenant e período (ver [data-model.md](./data-model.md)).
- **Item de fechamento**: linha de detalhe ligando uma entrega concluída ao valor de pagamento e de pedágio calculados para ela dentro de um fechamento.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O relatório de fechamento gerado pelo sistema é emitido sem qualquer ajuste manual e seus valores coincidem com a apuração equivalente feita fora do sistema, para o mesmo conjunto de entregas (spec-mãe, SC-004).
- **SC-002**: 100% das entregas concluídas dentro de um período aparecem no fechamento desse período, exceto as explicitamente sinalizadas como pendentes de configuração (FR-006).
- **SC-003**: Nenhuma tentativa de gerar um segundo fechamento para o mesmo tenant e período é bem-sucedida em produzir um resultado duplicado.
- **SC-004**: O tempo entre o encerramento do período e a disponibilização do fechamento é imediato (minutos), eliminando a apuração manual relatada como problema atual (documento de evidência, seção 1.2).

## Assumptions

- O período de fechamento (por padrão quinzenal, configurável por [HU11](../HU11-parametrizar-configuracoes-tenant/spec.md)) já está definido antes da geração — esta HU não define a periodicidade, apenas consome a configuração vigente.
- A geração do fechamento é acionada manualmente pelo administrador do tenant ao final do período; não há agendamento automático coberto por esta HU.
- Entregas sinalizadas como pendentes de configuração (FR-006) exigem correção manual do cadastro de tarifa/pedágio e reprocessamento — o mecanismo de reprocessamento não está detalhado nesta HU.
