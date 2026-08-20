---

description: "Task list for HU02 — Autoatribuir a entrega"
---

# Tasks: HU02 — Autoatribuir a entrega

**Input**: Design documents from `mile_bag_documentacao/specs/HU02-autoatribuir-entrega/`

**Prerequisites**: plan.md, spec.md, data-model.md, contracts/entrega-api.md, research.md, quickstart.md. Depende de HU01 já implementada (BDO confirmado).

**Tests**: Não solicitados no spec desta HU — nenhuma tarefa de teste incluída.

**Organization**: Esta HU tem uma única User Story (US1, P1) — todas as tarefas de implementação carregam o rótulo `[US1]`.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência de tarefa incompleta)
- **[US1]**: Tarefa da User Story 1 (única história desta HU)
- Caminhos de arquivo exatos incluídos em cada descrição

---

## Phase 1: Setup

**Purpose**: Estrutura inicial de pacotes para esta HU

- [ ] T001 Criar estrutura de pacotes `controller/`, `service/`, `repository/`, `model/` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência e isolamento multi-tenant que bloqueiam a implementação da história

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T002 Criar migração Flyway para a tabela `entrega` (`id`, `tenant_id`, `bdo_id` — FK única 1:1 com `bdo`, `motorista_responsavel` nullable, `status`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 [P] Criar entidade `Entrega` e o enum `EntregaStatus` (`AGUARDANDO_ATRIBUICAO`, `ATRIBUIDA`, `EM_ANDAMENTO`, `ENCERRADA_COM_SUCESSO`, `REABERTA` — ver `data-model.md`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/model/Entrega.java` e `EntregaStatus.java`, campos conforme `data-model.md`
- [ ] T004 Adicionar política de Row Level Security para isolamento por tenant na tabela `entrega`, na mesma migração de T002
- [ ] T005 Adicionar constraint de unicidade em `entrega.bdo_id`, garantindo relação 1:1 com o BDO de origem, na mesma migração de T002

**Checkpoint**: Fundação pronta — a User Story 1 pode ser implementada

---

## Phase 3: User Story 1 - Autoatribuir a entrega (Priority: P1) 🎯 MVP

**Goal**: O motorista atribui a si mesmo a entrega correspondente a um BDO confirmado, imediatamente após a confirmação, sem etapas intermediárias, e a entrega passa a constar na sua lista de entregas em andamento.

**Independent Test**: Capturar um BDO (HU01), confirmar, verificar que a opção de autoatribuição fica disponível e que a ação move a entrega para a lista do motorista.

### Implementation for User Story 1

- [ ] T006 [P] [US1] Implementar `EntregaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/repository/EntregaRepository.java`
- [ ] T007 [US1] Implementar `EntregaService.autoatribuir`, lendo o motorista responsável exclusivamente do claim do JWT autenticado (nunca de parâmetro do cliente) e validando unicidade da atribuição, em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/service/EntregaService.java` (depende de T006)
- [ ] T008 [US1] Adicionar validação: BDO não encontrado, não confirmado ou de outro tenant retorna `404`, em `EntregaService.java`
- [ ] T009 [US1] Adicionar validação: entrega já atribuída a outro motorista retorna `409` (Edge Case de dupla atribuição, FR-004), em `EntregaService.java`
- [ ] T010 [US1] Implementar endpoint `POST /api/v1/entregas/{bdoId}/autoatribuir` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/controller/EntregaController.java` (depende de T007–T009)
- [ ] T011 [US1] Implementar endpoint `GET /api/v1/entregas?motorista=me&status=EM_ANDAMENTO` (listagem das entregas do motorista autenticado) em `EntregaController.java` (depende de T007) — nota: `EM_ANDAMENTO` só é produzido depois de HU04 implementada
- [ ] T012 [P] [US1] Implementar ação de autoatribuição (botão + confirmação), encadeada à tela de conferência do BDO (HU01), em `mile_bag_app/src/features/motorista/captura-bdo/components/AutoatribuirEntrega.tsx`
- [ ] T013 [P] [US1] Implementar componente de listagem de entregas em andamento do motorista em `mile_bag_app/src/features/motorista/captura-bdo/components/ListaEntregasEmAndamento.tsx`
- [ ] T014 [US1] Implementar chamadas de API (autoatribuir, listagem) em `mile_bag_app/src/features/motorista/captura-bdo/services/entregaService.ts` (depende de T010, T011 — contrato de API)
- [ ] T015 [US1] Adicionar tratamento de erro `409` na UI (entrega já atribuída), exibindo mensagem clara ao motorista, em `AutoatribuirEntrega.tsx` (depende de T012, T014)

**Checkpoint**: A User Story 1 está funcional e testável de forma independente

---

## Phase Final: Polish & Cross-Cutting Concerns

- [ ] T016 [P] Adicionar logging estruturado das operações de autoatribuição em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/service/EntregaService.java`
- [ ] T017 Executar a validação de `quickstart.md` desta HU (`mile_bag_documentacao/specs/HU02-autoatribuir-entrega/quickstart.md`)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: depende da conclusão do Setup — bloqueia a User Story 1
- **User Story 1 (Phase 3)**: depende da conclusão da Phase 2 e da HU01 já implementada (BDO confirmado disponível)
- **Polish (Final Phase)**: depende da conclusão da User Story 1

### Within User Story 1

- Repositório (T006) antes do serviço (T007)
- Serviço e suas validações (T007–T009) antes dos endpoints (T010, T011)
- Componentes de UI (T012, T013) podem ser feitos em paralelo aos endpoints de back-end, mas a integração (T014, T015) depende do contrato de API já implementado

### Parallel Opportunities

- T003 (Foundational) não depende de outra tarefa da mesma fase
- T012 e T013 (componentes de UI) podem rodar em paralelo entre si
- T006 pode ser feita em paralelo a outras tarefas que não dependam dela

---

## Parallel Example: User Story 1

```bash
# Componentes de front-end em paralelo:
Task: "Implementar AutoatribuirEntrega.tsx em mile_bag_app/.../captura-bdo/components/"
Task: "Implementar ListaEntregasEmAndamento.tsx em mile_bag_app/.../captura-bdo/components/"
```

---

## Implementation Strategy

### MVP (única User Story desta HU)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (bloqueia a história)
3. Completar Phase 3: User Story 1
4. **Parar e validar**: rodar `quickstart.md` desta HU de forma isolada (requer HU01 já validada)
5. Entregar/demonstrar

### Observação

Esta HU introduz o pacote `business/entrega`, reaproveitado por todas as HUs seguintes do ciclo de entrega (HU03–HU08). Validar esta HU antes de iniciar HU03.

---

## Notes

- `[P]` = arquivos diferentes, sem dependências entre si
- `[US1]` mapeia a tarefa à única história desta HU, para rastreabilidade
- Sem tarefas de teste (não solicitadas no spec desta HU)
- Fazer commit após cada tarefa ou grupo lógico de tarefas
- Parar no checkpoint da Phase 3 para validar a história isoladamente antes de seguir para HU03
