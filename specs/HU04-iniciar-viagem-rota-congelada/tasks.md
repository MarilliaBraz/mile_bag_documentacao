---

description: "Task list for HU04 — Iniciar a viagem com a rota congelada"
---

# Tasks: HU04 — Iniciar a viagem com a rota congelada

**Input**: Design documents from `mile_bag_documentacao/specs/HU04-iniciar-viagem-rota-congelada/`

**Prerequisites**: plan.md, spec.md, data-model.md, contracts/viagem-api.md, research.md, quickstart.md. Depende de HU03 já implementada (rota selecionada para a entrega).

**Tests**: Não solicitados no spec desta HU — nenhuma tarefa de teste incluída.

**Organization**: Esta HU tem uma única User Story (US1, P1) — todas as tarefas de implementação carregam o rótulo `[US1]`.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência de tarefa incompleta)
- **[US1]**: Tarefa da User Story 1 (única história desta HU)
- Caminhos de arquivo exatos incluídos em cada descrição

---

## Phase 1: Setup

**Purpose**: Estrutura inicial de pacotes para esta HU

- [ ] T001 Criar estrutura de pacotes `controller/`, `service/`, `repository/`, `model/` em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência imutável e isolamento multi-tenant que bloqueiam a implementação da história

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T002 Criar migração Flyway para a tabela `viagem` (`id`, `entrega_id` — FK única, `km_ida`, `km_volta`, `pracas_pedagio` — JSON, `tempo_estimado`, `origem_selecao`, `data_inicio`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 [P] Criar entidade `Viagem` em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/model/Viagem.java`, imutável após a criação (sem setters para os campos congelados, conforme `data-model.md` e `../001-milebag-user-stories/research.md`, seção 7)
- [ ] T004 Adicionar política de Row Level Security para isolamento por tenant na tabela `viagem`, na mesma migração de T002
- [ ] T005 Adicionar constraint de unicidade em `viagem.entrega_id` (FR-004, uma viagem por entrega), na mesma migração de T002

**Checkpoint**: Fundação pronta — a User Story 1 pode ser implementada

---

## Phase 3: User Story 1 - Iniciar a viagem com a rota congelada (Priority: P1) 🎯 MVP

**Goal**: Ao iniciar a viagem, o sistema grava um snapshot imutável (quilometragem de ida/volta, pedágios, tempo estimado) a partir da rota selecionada em HU03, que passa a ser a base exclusiva do cálculo de pagamento.

**Independent Test**: Com uma rota já selecionada (HU03), iniciar a viagem e verificar que o snapshot gravado não muda mesmo que os dados de mapa mudem depois.

### Implementation for User Story 1

- [ ] T006 [P] [US1] Implementar `ViagemRepository`, sem métodos de atualização expostos (a ausência do `UPDATE` é o mecanismo de imutabilidade — ver `contracts/viagem-api.md`), em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/repository/ViagemRepository.java`
- [ ] T007 [US1] Implementar `ViagemService.iniciarViagem`, lendo a última seleção de rota registrada pela entrega (`PropostaRota` de HU03), congelando os valores no snapshot e, na mesma transação, atualizando `Entrega.status` para `EntregaStatus.EM_ANDAMENTO` (enum canônico de `../HU02-autoatribuir-entrega/data-model.md`), em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/service/ViagemService.java` (depende de T006)
- [ ] T008 [US1] Adicionar validação: entrega sem rota selecionada retorna `422` (FR-001), em `ViagemService.java`
- [ ] T009 [US1] Adicionar validação: entrega que já possui viagem iniciada retorna `409` (FR-004), em `ViagemService.java`
- [ ] T010 [US1] Implementar sinalização, no snapshot, de trechos de pedágio identificados sem valor cadastrado, sem bloquear a criação (FR-006, Edge Case), em `ViagemService.java`
- [ ] T011 [US1] Implementar endpoint `POST /api/v1/entregas/{entregaId}/viagem/iniciar` em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/controller/ViagemController.java` (depende de T007–T010)
- [ ] T012 [US1] Implementar endpoint `GET /api/v1/entregas/{entregaId}/viagem` (consulta do snapshot) em `ViagemController.java` (depende de T006)
- [ ] T013 [P] [US1] Implementar ação "iniciar viagem", encadeada à seleção de rota de HU03, em `mile_bag_app/src/features/motorista/rota/components/IniciarViagem.tsx`
- [ ] T014 [US1] Implementar chamada de API (iniciar viagem, consultar snapshot) em `mile_bag_app/src/features/motorista/rota/services/viagemService.ts` (depende de T011, T012 — contrato de API)
- [ ] T015 [US1] Implementar exibição, somente leitura, do snapshot congelado após o início da viagem em `mile_bag_app/src/features/motorista/rota/components/ResumoViagem.tsx` (depende de T013, T014)

**Checkpoint**: A User Story 1 está funcional e testável de forma independente

---

## Phase Final: Polish & Cross-Cutting Concerns

- [ ] T016 [P] Adicionar logging estruturado do início de viagem e de qualquer tentativa de alteração do snapshot (deve ser rejeitada) em `mile_bag_api/src/main/java/com/marillia/milebag/business/viagem/service/ViagemService.java`
- [ ] T017 Executar a validação de `quickstart.md` desta HU (`mile_bag_documentacao/specs/HU04-iniciar-viagem-rota-congelada/quickstart.md`), incluindo o teste de imutabilidade (alterar dado externo e confirmar que o snapshot não muda)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: depende da conclusão do Setup — bloqueia a User Story 1
- **User Story 1 (Phase 3)**: depende da conclusão da Phase 2 e da HU03 já implementada (rota selecionada disponível)
- **Polish (Final Phase)**: depende da conclusão da User Story 1

### Within User Story 1

- Repositório (T006) antes do serviço (T007)
- Serviço e validações (T007–T010) antes dos endpoints (T011, T012)
- Componente de UI (T013) pode ser feito em paralelo aos endpoints de back-end, mas a integração (T014, T015) depende do contrato de API já implementado

### Parallel Opportunities

- T003 (Foundational) não depende de outra tarefa da mesma fase
- T013 pode ser desenvolvido em paralelo às tarefas de back-end (T006–T012)

---

## Parallel Example: User Story 1

```bash
# Repositório (sem update) e componente de UI podem avançar em paralelo:
Task: "Implementar ViagemRepository em mile_bag_api/.../business/viagem/repository/ViagemRepository.java"
Task: "Implementar IniciarViagem.tsx em mile_bag_app/.../rota/components/"
```

---

## Implementation Strategy

### MVP (única User Story desta HU)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (bloqueia a história)
3. Completar Phase 3: User Story 1
4. **Parar e validar**: rodar `quickstart.md` desta HU de forma isolada (requer HU03 já validada), com atenção especial ao teste de imutabilidade
5. Entregar/demonstrar — com esta HU concluída, o núcleo do problema central do projeto (rastreabilidade do cálculo de pagamento) está resolvido

### Observação

Esta HU fecha o núcleo de execução da entrega antes do comprovante (HU06). O snapshot aqui criado é consumido, sem recálculo, pelo fechamento financeiro (User Story 2 do spec guarda-chuva, fora do escopo desta HU).

---

## Notes

- `[P]` = arquivos diferentes, sem dependências entre si
- `[US1]` mapeia a tarefa à única história desta HU, para rastreabilidade
- Sem tarefas de teste (não solicitadas no spec desta HU)
- Fazer commit após cada tarefa ou grupo lógico de tarefas
- Parar no checkpoint da Phase 3 para validar a história isoladamente, com foco na imutabilidade do snapshot, antes de seguir para HU05/HU06
