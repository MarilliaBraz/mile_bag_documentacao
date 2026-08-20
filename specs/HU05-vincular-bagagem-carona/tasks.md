---

description: "Task list for HU05 — Vincular bagagem carona à entrega"
---

# Tasks: Vincular bagagem carona à entrega

**Input**: Design documents from `mile_bag_documentacao/specs/HU05-vincular-bagagem-carona/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/bagagem-carona-api.md](./contracts/bagagem-carona-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organização**: Esta HU tem uma única User Story (US1, P1). Fases: Setup → Foundational → US1 → Polish.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 (única história desta HU)

## Path Conventions

- Back-end: `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/bagagemcarona/`
- Front-end (PWA motorista): `mile_bag_app/src/features/motorista/captura-bdo/bagagem-carona/`

---

## Phase 1: Setup

**Purpose**: Nenhuma dependência ou ferramenta nova é exigida por esta HU além do já estabelecido no plano-mãe (`../001-milebag-user-stories/plan.md`).

- [ ] T001 Confirmar que o pacote `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/` já existe (criado pela implementação de HU01–HU04); se não existir, criar o pacote base antes de prosseguir

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Pré-requisitos que bloqueiam a implementação da User Story 1 desta HU

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T002 Criar migração Flyway para a tabela `bagagem_carona` (colunas: `id`, `bdo_id`, `entrega_principal_id`, `quilometragem_complementar`, `criado_em`, coluna de tenant, política de Row Level Security) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 Confirmar que a entidade `Entrega` (de HU02) expõe o campo `status` (`EntregaStatus` — ver `../HU02-autoatribuir-entrega/data-model.md`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/model/Entrega.java` — se não existir, adicionar

**Checkpoint**: Schema e status de Entrega prontos — implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Vincular bagagem carona à entrega em andamento (Priority: P1) 🎯 MVP

**Goal**: Permitir que o motorista vincule, durante uma entrega em andamento, uma ou mais bagagens carona a partir de BDOs próprios, remunerando apenas a quilometragem complementar, sem abrir uma viagem separada.

**Independent Test**: Com uma entrega principal já autoatribuída e em andamento (HU02), capturar o BDO de uma segunda bagagem e vinculá-la a essa entrega via `POST /api/v1/entregas/{entregaId}/bagagens-carona`, sem depender de rota calculada ou comprovante já registrado.

### Implementation for User Story 1

- [ ] T004 [P] [US1] Criar entidade JPA `BagagemCarona` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/bagagemcarona/BagagemCarona.java`, com os campos de `data-model.md` (`id`, `bdoId`, `entregaPrincipalId`, `quilometragemComplementar`, `criadoEm`)
- [ ] T005 [P] [US1] Criar `BagagemCaronaRepository` (Spring Data JPA) em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/bagagemcarona/BagagemCaronaRepository.java`
- [ ] T006 [US1] Implementar `BagagemCaronaService.vincular(entregaId, bdoId)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/bagagemcarona/BagagemCaronaService.java`: valida que a entrega principal está em andamento (rejeita com 409 se `ENCERRADA_COM_SUCESSO`, FR-002), valida que o `bdoId` não está vinculado a outra bagagem carona (rejeita com 409, FR-003), calcula e congela `quilometragemComplementar` (depende de T004, T005)
- [ ] T007 [US1] Implementar `BagagemCaronaService.desvincular(entregaId, bagagemCaronaId)`: remove o vínculo apenas se a entrega principal ainda não estiver `ENCERRADA_COM_SUCESSO` (edge case do spec.md), retorna 409 caso contrário (depende de T006)
- [ ] T008 [US1] Implementar `BagagemCaronaService.listarPorEntrega(entregaId)` para uso pela tela do motorista e pelo fechamento (HU12) (depende de T004, T005)
- [ ] T009 [US1] Implementar endpoint `POST /api/v1/entregas/{entregaId}/bagagens-carona` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/bagagemcarona/BagagemCaronaController.java`, conforme `contracts/bagagem-carona-api.md` (depende de T006)
- [ ] T010 [US1] Implementar endpoint `DELETE /api/v1/entregas/{entregaId}/bagagens-carona/{bagagemCaronaId}` no mesmo controller, conforme `contracts/bagagem-carona-api.md` (depende de T007)
- [ ] T011 [US1] Implementar endpoint `GET /api/v1/entregas/{entregaId}/bagagens-carona` no mesmo controller, conforme `contracts/bagagem-carona-api.md` (depende de T008)
- [ ] T012 [US1] Garantir que a reabertura de uma entrega por ocorrência (HU07) não remove nem altera as `BagagemCarona` já vinculadas (FR-005) — ajustar `EntregaService.reabrir(...)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/EntregaService.java` se necessário (depende de T006)
- [ ] T013 [P] [US1] Criar tela/ação "Vincular bagagem carona" na feature do motorista em `mile_bag_app/src/features/motorista/captura-bdo/bagagem-carona/VincularBagagemCarona.tsx`, acionável a partir de uma entrega em andamento
- [ ] T014 [US1] Integrar a chamada ao `POST /api/v1/entregas/{entregaId}/bagagens-carona` no client de API em `mile_bag_app/src/features/motorista/captura-bdo/bagagem-carona/bagagemCaronaService.ts` (depende de T013)
- [ ] T015 [US1] Garantir que a ação de vincular bagagem carona funcione na fila offline (mesma infraestrutura de HU08) — registrar o evento localmente quando sem conexão, em `mile_bag_app/src/features/motorista/offline-sync/` (depende de T014; integra com HU08)
- [ ] T016 [US1] Adicionar tratamento de erro na UI para os casos 404/409 (entrega encerrada, BDO já vinculado) em `mile_bag_app/src/features/motorista/captura-bdo/bagagem-carona/VincularBagagemCarona.tsx` (depende de T014)

**Checkpoint**: Neste ponto, a User Story 1 desta HU deve estar completa e testável de forma independente via `quickstart.md`

---

## Phase 4: Polish & Cross-Cutting Concerns

- [ ] T017 [P] Adicionar teste automatizado de isolamento entre tenants para `BagagemCarona` (RNF01), seguindo o padrão já usado nas demais entidades do projeto
- [ ] T018 Run quickstart.md validation — executar o roteiro de [`quickstart.md`](./quickstart.md) de ponta a ponta

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: Depende da conclusão do Setup — BLOQUEIA a User Story 1
- **User Story 1 (Phase 3)**: Depende da conclusão da Phase 2
- **Polish (Phase 4)**: Depende da conclusão da Phase 3

### Within User Story 1

- Entidade e repositório (T004, T005) antes do serviço (T006)
- Serviço (T006–T008) antes dos endpoints (T009–T011)
- Backend (T004–T012) antes da integração de front-end que os consome (T013–T016), mas a criação da tela (T013) pode começar em paralelo
- Integração offline (T015) depende da tela e do client de API já existirem (T013, T014)

### Parallel Opportunities

- T004 e T005 podem rodar em paralelo (arquivos diferentes)
- T013 pode começar em paralelo ao trabalho de backend (T004–T012), já que é a criação da tela sem lógica de chamada ainda
- T017 (teste de isolamento) pode rodar em paralelo a T018 (validação do quickstart)

---

## Parallel Example: User Story 1

```bash
# Lançar em paralelo:
Task: "Criar entidade JPA BagagemCarona em mile_bag_api/.../bagagemcarona/BagagemCarona.java"
Task: "Criar BagagemCaronaRepository em mile_bag_api/.../bagagemcarona/BagagemCaronaRepository.java"
Task: "Criar tela VincularBagagemCarona.tsx em mile_bag_app/.../bagagem-carona/"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRÍTICO — bloqueia a User Story)
3. Completar Phase 3: User Story 1
4. **PARAR e VALIDAR**: testar a User Story 1 de forma independente via `quickstart.md`
5. Completar Phase 4: Polish

### Notas

- Esta HU depende funcionalmente de HU01–HU04 (entrega já autoatribuída e em andamento) e integra-se com HU06 (encerramento com comprovante), HU07 (reabertura por ocorrência), HU08 (fila offline) e HU12 (fechamento) — mas é implementável e testável de forma independente, desde que essas dependências já existam no ambiente de teste.
- Commitar após cada tarefa ou grupo lógico de tarefas.
- Evitar: tarefas vagas, conflito de mesmo arquivo entre tarefas paralelas, dependências cruzadas que quebrem a testabilidade independente da história.
