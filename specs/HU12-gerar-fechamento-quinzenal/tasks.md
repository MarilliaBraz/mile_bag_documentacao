---

description: "Task list for HU12 — Gerar o fechamento quinzenal"
---

# Tasks: HU12 — Gerar o fechamento quinzenal

**Input**: Design documents from `mile_bag_documentacao/specs/HU12-gerar-fechamento-quinzenal/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/fechamento-api.md](./contracts/fechamento-api.md), [quickstart.md](./quickstart.md)

**Pré-requisito de outra feature**: `TabelaTarifaService.resolverTarifaAplicavel` de [HU09](../HU09-configurar-tabelas-tarifa/tasks.md#T008) e `PracaPedagioService.calcularReembolso`/`valorVigenteEm` de [HU10](../HU10-cadastrar-pracas-pedagio/tasks.md#T011) MUST estar implementados antes de iniciar a Phase 3 desta HU — ver seção "Dependência entre features" abaixo.

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organization**: HU12 tem uma única User Story (P1); todas as tarefas de implementação carregam o rótulo `[US1]`.

## Phase 1: Setup

- [ ] T001 Confirmar/criar o subpacote `business/fechamento` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/` (`controller/`, `service/`, `repository/`, `model/`)
- [ ] T002 Confirmar que as dependências de geração de PDF (Apache PDFBox ou JasperReports Library, decisão em [research.md do plano-mãe](../001-milebag-user-stories/research.md)) estão no `pom.xml` de `mile_bag_api`

## Phase 2: Foundational

**Purpose**: Schema de `Fechamento`/`ItemFechamento`, unicidade por período e isolamento por tenant, bloqueantes para a User Story

- [ ] T003 Criar migração Flyway para a tabela `fechamento` (`id`, `tenant_id`, `periodo_inicio`, `periodo_fim`, `gerado_em`, `status`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T004 Adicionar constraint de unicidade `(tenant_id, periodo_inicio, periodo_fim)` na tabela `fechamento`, na mesma migração de T003 (FR-005)
- [ ] T005 Criar migração Flyway para a tabela `item_fechamento` (`id`, `fechamento_id`, `entrega_id`, `motorista_id`, `valor_pagamento_entrega`, `valor_reembolso_pedagio`, `status_item`, `motivo_pendencia`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T006 Adicionar política de Row Level Security para `fechamento` filtrando por `tenant_id` (a tabela `item_fechamento` herda isolamento via join com `fechamento`) em `mile_bag_api/src/main/resources/db/migration/`

**Checkpoint**: Schema de fechamento e isolamento prontos — a implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Gerar o fechamento periódico de pagamento (Priority: P1) 🎯 MVP

**Goal**: Calcular, para um período, o valor de pagamento de entregas e o valor de reembolso de pedágios devidos a cada motorista, a partir de entregas concluídas com snapshot de rota congelado, sem intervenção manual e sem duplicar fechamentos do mesmo período.

**Independent Test**: Com um tenant que já tenha entregas concluídas com snapshot de rota congelado (dependência de HU04 do spec-mãe) e tarifas/pedágios cadastrados (HU09, HU10), gerar o fechamento do período e conferir os totais por motorista contra um cálculo manual de referência.

### Implementation for User Story 1

- [ ] T007 [P] [US1] Criar entidade `Fechamento` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/model/Fechamento.java` (`id`, `tenantId`, `periodoInicio`, `periodoFim`, `geradoEm`, `status` enum `StatusFechamento` [`COMPLETO`, `COM_PENDENCIAS`])
- [ ] T008 [P] [US1] Criar entidade `ItemFechamento` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/model/ItemFechamento.java` (`id`, `fechamentoId`, `entregaId`, `motoristaId`, `valorPagamentoEntrega`, `valorReembolsoPedagio`, `statusItem` enum `StatusItemFechamento` [`CALCULADO`, `PENDENTE`], `motivoPendencia`)
- [ ] T009 [P] [US1] Criar `FechamentoRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/repository/FechamentoRepository.java` com busca por `tenantId`, e por `(tenantId, periodoInicio, periodoFim)` para checagem de idempotência (FR-005)
- [ ] T010 [P] [US1] Criar `ItemFechamentoRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/repository/ItemFechamentoRepository.java` com busca por `fechamentoId`
- [ ] T011 [US1] Implementar `FechamentoService.buscarEntregasElegiveis(tenantId, periodoInicio, periodoFim)`, retornando as entregas concluídas com comprovante dentro do período (FR-004), excluindo entregas ainda em andamento e considerando apenas a conclusão que ocorreu dentro do período mesmo após reaberturas por ocorrência — depende de T007–T010
- [ ] T012 [US1] Implementar `FechamentoService.calcularItem(entrega)`: resolve a tarifa aplicável via `TabelaTarifaService.resolverTarifaAplicavel` ([HU09](../HU09-configurar-tabelas-tarifa/tasks.md#T008)) e calcula `valorPagamentoEntrega` a partir da quilometragem do snapshot da entrega; resolve `valorReembolsoPedagio` a partir dos valores de pedágio já congelados no snapshot (não uma nova consulta a `PracaPedagioService`); marca `statusItem = PENDENTE` com `motivoPendencia` quando faltar tarifa ou valor de pedágio resolvível (FR-006), sem interromper o cálculo dos demais itens — depende de T011
- [ ] T013 [US1] Implementar `FechamentoService.gerar(tenantId, periodoInicio, periodoFim)`: verifica idempotência via `FechamentoRepository` (retorna o fechamento existente em vez de recriar — FR-005), senão cria o `Fechamento`, itera as entregas elegíveis aplicando T012 a cada uma, persiste os `ItemFechamento`, e define `status = COM_PENDENCIAS` se qualquer item estiver `PENDENTE`, senão `COMPLETO` — depende de T012
- [ ] T014 [US1] Implementar em `FechamentoService` os métodos de consulta: listar fechamentos do tenant, detalhe de um fechamento com seus itens, escopados ao tenant autenticado (FR-008) — depende de T013
- [ ] T015 [US1] Implementar `RelatorioFechamentoService` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/service/RelatorioFechamentoService.java`, gerando dois PDFs distintos (pagamento de entregas e reembolso de pedágios, FR-003/RN08) a partir dos `ItemFechamento` de um fechamento, usando a biblioteca definida em T002 — depende de T014
- [ ] T016 [US1] Criar `FechamentoController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/controller/FechamentoController.java` implementando os 5 endpoints de [contracts/fechamento-api.md](./contracts/fechamento-api.md) (`POST /api/v1/fechamentos`, `GET /api/v1/fechamentos`, `GET /api/v1/fechamentos/{id}`, `GET /api/v1/fechamentos/{id}/relatorio-pagamento-entregas.pdf`, `GET /api/v1/fechamentos/{id}/relatorio-reembolso-pedagios.pdf`) — depende de T013, T014, T015
- [ ] T017 [US1] Mapear as respostas do controller: `201` (fechamento novo), `200` (fechamento já existente para o mesmo período, reaproveitado), `409` (período se sobrepõe parcialmente a um fechamento existente sem coincidir exatamente) — depende de T016
- [ ] T018 [P] [US1] Criar tela de fechamento em `mile_bag_app/src/features/back-office/fechamento/` (botão de gerar fechamento do período, listagem de fechamentos anteriores, detalhe com itens e destaque visual para itens `PENDENTE`, links de download dos dois PDFs) — depende de T016

**Checkpoint**: User Story 1 (HU12) completa e testável de forma independente — administrador do tenant consegue gerar o fechamento de um período, com totais de pagamento e de pedágio separados, sem duplicação e sem intervenção manual

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T019 [P] Adicionar logging da geração de fechamento (incluindo itens marcados `PENDENTE`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/service/FechamentoService.java`
- [ ] T020 Executar a validação de [quickstart.md](./quickstart.md) (geração do fechamento, conferência dos totais, tentativa de duplicação, item pendente por falta de tarifa/pedágio)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências.
- **Foundational (Phase 2)**: Depende do Setup — BLOQUEIA a User Story 1.
- **User Story 1 (Phase 3)**: Depende da conclusão da Fase 2 **e** da implementação prévia de HU09 e HU10 (ver abaixo).
- **Polish (Final Phase)**: Depende da conclusão da Fase 3.

### Dependência entre features

Esta HU **não é independente**: T012 chama diretamente `TabelaTarifaService.resolverTarifaAplicavel` ([HU09](../HU09-configurar-tabelas-tarifa/tasks.md), tarefa T008 de lá) e consome os valores de pedágio já resolvidos por `PracaPedagioService` ([HU10](../HU10-cadastrar-pracas-pedagio/tasks.md), tarefas T011/T012 de lá). A Phase 3 desta HU só pode começar depois que HU09 e HU10 estiverem, no mínimo, com seus respectivos Foundational + serviços de resolução (T007 de HU09; T011 de HU10) implementados. A periodicidade do período de fechamento é consumida de HU11 (`PeriodicidadeFechamentoConfigService.periodicidadeVigenteEm`), mas essa integração pode usar o valor default `QUINZENAL` até HU11 estar disponível.

### Within User Story 1

- Modelos (T007, T008) e repositórios (T009, T010) antes do serviço (T011–T015).
- `buscarEntregasElegiveis` (T011) antes de `calcularItem` (T012), que antes de `gerar` (T013).
- Serviço de geração (T013) antes das consultas (T014) e antes do serviço de relatório (T015).
- Serviço completo antes do controller (T016–T017).
- Back-end antes da tela de front-end (T018).

### Parallel Opportunities

- T007, T008, T009, T010 podem rodar em paralelo (arquivos diferentes).
- T018 pode começar em paralelo ao back-end assim que o contrato de API estiver estável, mas a integração final depende de T016.

---

## Parallel Example: User Story 1

```bash
# Modelos e repositórios em paralelo:
Task: "Criar entidade Fechamento em mile_bag_api/.../model/Fechamento.java"
Task: "Criar entidade ItemFechamento em mile_bag_api/.../model/ItemFechamento.java"
Task: "Criar FechamentoRepository em mile_bag_api/.../repository/FechamentoRepository.java"
Task: "Criar ItemFechamentoRepository em mile_bag_api/.../repository/ItemFechamentoRepository.java"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Confirmar que HU09 e HU10 já concluíram, no mínimo, seus serviços de resolução de tarifa e de pedágio.
2. Completar Phase 1: Setup.
3. Completar Phase 2: Foundational (schema + unicidade + RLS) — bloqueante.
4. Completar Phase 3: User Story 1.
5. **Parar e validar**: rodar [quickstart.md](./quickstart.md) de ponta a ponta.
6. Prosseguir para HU13, que aplica a identidade visual aos relatórios já gerados por T015.

---

## Notes

- `[P]` = arquivos diferentes, sem dependência entre as tarefas marcadas.
- `[US1]` mapeia a tarefa à única User Story desta HU.
- Commit após cada tarefa ou grupo lógico coeso.
- Evitar: tarefas vagas, conflito no mesmo arquivo, dependências que quebrem a testabilidade independente da história (exceto a dependência explícita e documentada com HU09/HU10 acima).
