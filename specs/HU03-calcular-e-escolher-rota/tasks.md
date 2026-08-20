---

description: "Task list for HU03 — Calcular e escolher a rota da entrega"
---

# Tasks: HU03 — Calcular e escolher a rota da entrega

**Input**: Design documents from `mile_bag_documentacao/specs/HU03-calcular-e-escolher-rota/`

**Prerequisites**: plan.md, spec.md, data-model.md, contracts/rota-api.md, research.md, quickstart.md. Depende de HU02 já implementada (entrega autoatribuída).

**Tests**: Não solicitados no spec desta HU — nenhuma tarefa de teste incluída.

**Organization**: Esta HU tem uma única User Story (US1, P1) — todas as tarefas de implementação carregam o rótulo `[US1]`.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência de tarefa incompleta)
- **[US1]**: Tarefa da User Story 1 (única história desta HU)
- Caminhos de arquivo exatos incluídos em cada descrição

---

## Phase 1: Setup

**Purpose**: Estrutura inicial de pacotes/feature e clientes de serviços externos de rota/geocodificação

- [ ] T001 Criar estrutura de pacotes `controller/`, `service/`, `repository/`, `model/` em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/`
- [ ] T002 [P] Criar estrutura de pastas `components/`, `hooks/`, `services/`, `types/` em `mile_bag_app/src/features/motorista/rota/`
- [ ] T003 [P] Configurar cliente HTTP para o serviço de rotas open source (Valhalla, conforme decisão default em `../001-milebag-user-stories/research.md`, seção 5) em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/service/RoutingClient.java` (esqueleto de configuração, sem lógica de negócio)
- [ ] T004 [P] Criar componente de mapa compartilhado (Leaflet sobre OpenStreetMap, com atribuição visível — RI08) em `mile_bag_app/src/shared/mapa/MapaBase.tsx`, reutilizável por esta HU e por HU04

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência e isolamento multi-tenant que bloqueiam a implementação da história

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T005 Criar migração Flyway para a tabela `proposta_rota` (`entrega_id` — FK única, substituída a cada novo cálculo; `origem`, `destino`, `alternativas` — JSON, `tipo_selecao`, `justificativa`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T006 [P] Criar entidade `PropostaRota` em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/model/PropostaRota.java`, campos conforme `data-model.md`
- [ ] T007 Adicionar política de Row Level Security para isolamento por tenant na tabela `proposta_rota` (via `tenant_id` ou junção com `entrega`), na mesma migração de T005

**Checkpoint**: Fundação pronta — a User Story 1 pode ser implementada

---

## Phase 3: User Story 1 - Calcular e escolher a rota da entrega (Priority: P1) 🎯 MVP

**Goal**: O motorista vê rotas alternativas (distância, tempo, pedágios) até o endereço de entrega e escolhe uma delas ou traça uma rota própria com justificativa.

**Independent Test**: Com uma entrega já autoatribuída (HU02), solicitar o cálculo de rota e validar que distância, tempo e pedágios aparecem corretamente para o endereço do BDO.

### Implementation for User Story 1

- [ ] T008 [P] [US1] Implementar `PropostaRotaRepository`, com upsert por `entregaId` (cada novo cálculo substitui o anterior — Edge Case do spec), em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/repository/PropostaRotaRepository.java`
- [ ] T009 [US1] Implementar `RoutingClient`, integrando o serviço de rotas (Valhalla/OSRM) para cálculo de distância, tempo estimado e trechos com pedágio, em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/service/RoutingClient.java` (depende de T003)
- [ ] T010 [US1] Implementar `GeocodingClient`, integrando o serviço de geocodificação (Nominatim/Photon) para resolver o endereço de entrega do BDO, em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/service/GeocodingClient.java`
- [ ] T011 [US1] Implementar `RotaService`, orquestrando geocodificação, cálculo de alternativas e persistência (substituindo cálculo anterior da mesma entrega), em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/service/RotaService.java` (depende de T008, T009, T010)
- [ ] T012 [US1] Implementar endpoint `POST /api/v1/entregas/{entregaId}/rotas/calcular`, aceitando coordenadas manuais opcionais de destino (FR-006), em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/controller/RotaController.java` (depende de T011)
- [ ] T013 [US1] Implementar endpoint `POST /api/v1/entregas/{entregaId}/rotas/selecionar`, validando `justificativa` obrigatória quando `tipoSelecao = propria` (FR-005), em `RotaController.java` (depende de T011)
- [ ] T014 [US1] Adicionar validação: endereço não geocodificável sem coordenada manual retorna `422`, em `RotaService.java`
- [ ] T015 [P] [US1] Implementar exibição das rotas alternativas no mapa (distância, tempo, pedágios) em `mile_bag_app/src/features/motorista/rota/components/MapaRotas.tsx` (usa `shared/mapa/MapaBase.tsx`)
- [ ] T016 [P] [US1] Implementar UI de traçado de rota própria com campo de justificativa obrigatório em `mile_bag_app/src/features/motorista/rota/components/RotaPropria.tsx`
- [ ] T017 [US1] Implementar UI de ajuste manual do ponto de destino quando a geocodificação falhar (Edge Case) em `mile_bag_app/src/features/motorista/rota/components/AjusteDestinoManual.tsx`
- [ ] T018 [US1] Implementar chamadas de API (calcular, selecionar) em `mile_bag_app/src/features/motorista/rota/services/rotaService.ts` (depende de T012, T013 — contrato de API)
- [ ] T019 [US1] Conectar o fluxo cálculo → seleção em uma única tela (SC-002), em `mile_bag_app/src/features/motorista/rota/hooks/useRota.ts` (depende de T015, T016, T017, T018)

**Checkpoint**: A User Story 1 está funcional e testável de forma independente

---

## Phase Final: Polish & Cross-Cutting Concerns

- [ ] T020 [P] Adicionar logging estruturado das operações de cálculo e seleção de rota em `mile_bag_api/src/main/java/com/marillia/milebag/business/rota/service/RotaService.java`
- [ ] T021 Executar a validação de `quickstart.md` desta HU (`mile_bag_documentacao/specs/HU03-calcular-e-escolher-rota/quickstart.md`)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: depende da conclusão do Setup — bloqueia a User Story 1
- **User Story 1 (Phase 3)**: depende da conclusão da Phase 2 e da HU02 já implementada (entrega autoatribuída disponível)
- **Polish (Final Phase)**: depende da conclusão da User Story 1

### Within User Story 1

- Repositório (T008) e clientes externos (T009, T010) antes do serviço de orquestração (T011)
- Serviço e validações (T011, T014) antes dos endpoints (T012, T013)
- Componentes de UI (T015–T017) podem ser feitos em paralelo aos endpoints de back-end, mas a integração (T018, T019) depende do contrato de API já implementado

### Parallel Opportunities

- T002, T003, T004 (Setup) podem rodar em paralelo com T001
- T009 e T010 (clientes de rotas e geocodificação) podem rodar em paralelo entre si
- T015 e T016 (componentes de UI) podem rodar em paralelo entre si

---

## Parallel Example: User Story 1

```bash
# Clientes de serviços externos em paralelo:
Task: "Implementar RoutingClient em mile_bag_api/.../business/rota/service/RoutingClient.java"
Task: "Implementar GeocodingClient em mile_bag_api/.../business/rota/service/GeocodingClient.java"

# Componentes de front-end em paralelo:
Task: "Implementar MapaRotas.tsx em mile_bag_app/.../rota/components/"
Task: "Implementar RotaPropria.tsx em mile_bag_app/.../rota/components/"
```

---

## Implementation Strategy

### MVP (única User Story desta HU)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (bloqueia a história)
3. Completar Phase 3: User Story 1
4. **Parar e validar**: rodar `quickstart.md` desta HU de forma isolada (requer HU02 já validada)
5. Entregar/demonstrar

### Observação

Esta HU é pré-requisito direto de HU04 (congelamento da rota) — a Proposta de Rota selecionada aqui é o insumo do snapshot imutável de Viagem. Validar esta HU antes de iniciar HU04.

---

## Notes

- `[P]` = arquivos diferentes, sem dependências entre si
- `[US1]` mapeia a tarefa à única história desta HU, para rastreabilidade
- Sem tarefas de teste (não solicitadas no spec desta HU)
- Fazer commit após cada tarefa ou grupo lógico de tarefas
- Parar no checkpoint da Phase 3 para validar a história isoladamente antes de seguir para HU04
