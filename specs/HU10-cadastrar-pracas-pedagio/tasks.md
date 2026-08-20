---

description: "Task list for HU10 — Cadastrar praças de pedágio"
---

# Tasks: HU10 — Cadastrar praças de pedágio

**Input**: Design documents from `mile_bag_documentacao/specs/HU10-cadastrar-pracas-pedagio/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/pedagios-api.md](./contracts/pedagios-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organization**: HU10 tem uma única User Story (P1); todas as tarefas de implementação carregam o rótulo `[US1]`.

## Phase 1: Setup

- [ ] T001 Confirmar/criar o subpacote `business/pedagio` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/` (`controller/`, `service/`, `repository/`, `model/`)

## Phase 2: Foundational

**Purpose**: Schema de dados histórico (praça + valores) e isolamento por tenant, bloqueantes para a User Story

- [ ] T002 Criar migração Flyway para a tabela `praca_pedagio` (`id`, `tenant_id`, `nome`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 Criar migração Flyway para a tabela `praca_pedagio_valor` (`id`, `praca_pedagio_id`, `valor`, `vigente_desde`) na mesma migração ou subsequente, com índice em `(praca_pedagio_id, vigente_desde)` para acelerar a resolução do valor vigente em uma data
- [ ] T004 Adicionar política de Row Level Security para `praca_pedagio` filtrando por `tenant_id` (a tabela `praca_pedagio_valor` herda o isolamento via join com `praca_pedagio`) em `mile_bag_api/src/main/resources/db/migration/`

**Checkpoint**: Schema histórico e isolamento prontos — a implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Cadastrar praças de pedágio do tenant (Priority: P1) 🎯 MVP

**Goal**: Permitir que o administrador do tenant cadastre praças de pedágio com histórico de valores versionado por data de vigência, sem nunca sobrescrever valores anteriores, e disponibilizar a consulta do valor vigente em uma data para o congelamento de rota e o fechamento.

**Independent Test**: Cadastrar uma praça com valor e data de vigência, atualizar esse valor posteriormente, e conferir que o histórico de valores anteriores permanece consultável — sem depender do cálculo de rota nem do fechamento estarem implementados.

### Implementation for User Story 1

- [ ] T005 [P] [US1] Criar entidade `PracaPedagio` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/model/PracaPedagio.java` com os campos `id`, `tenantId`, `nome`
- [ ] T006 [P] [US1] Criar entidade `PracaPedagioValor` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/model/PracaPedagioValor.java` com os campos `id`, `pracaPedagioId`, `valor`, `vigenteDesde`
- [ ] T007 [P] [US1] Criar `PracaPedagioRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/repository/PracaPedagioRepository.java` com busca por `tenantId`
- [ ] T008 [P] [US1] Criar `PracaPedagioValorRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/repository/PracaPedagioValorRepository.java` com método para buscar o valor de maior `vigenteDesde` que seja `<=` a uma data informada, para uma dada praça (usado por T011)
- [ ] T009 [US1] Implementar `PracaPedagioService.criarPraca(...)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/service/PracaPedagioService.java`, criando a praça e seu primeiro `PracaPedagioValor` (FR-001) — depende de T005–T008
- [ ] T010 [US1] Implementar `PracaPedagioService.adicionarValor(pracaId, valor, vigenteDesde)`, rejeitando (`400`) quando `vigenteDesde` for anterior ou igual à `vigenteDesde` do valor vigente mais recente (FR-005) — depende de T009
- [ ] T011 [US1] Implementar `PracaPedagioService.valorVigenteEm(pracaId, data)`, retornando o valor de maior `vigenteDesde` `<=` à data informada, ou erro `404` se não houver nenhum (FR-003) — depende de T008
- [ ] T012 [US1] Implementar `PracaPedagioService.calcularReembolso(pracaId, data)` retornando `2 × valorVigenteEm(pracaId, data)` (ida + volta, FR-004) — depende de T011
- [ ] T013 [US1] Implementar métodos de listagem (praças com valor vigente atual) e histórico de valores no `PracaPedagioService`, escopados ao tenant autenticado (FR-006) — depende de T009
- [ ] T014 [US1] Criar `PracaPedagioController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/controller/PracaPedagioController.java` implementando os 5 endpoints de [contracts/pedagios-api.md](./contracts/pedagios-api.md) (`POST /api/v1/pracas-pedagio`, `GET /api/v1/pracas-pedagio`, `GET /api/v1/pracas-pedagio/{id}/valores`, `POST /api/v1/pracas-pedagio/{id}/valores`, `GET /api/v1/pracas-pedagio/{id}/valor-em`) — depende de T010, T011, T013
- [ ] T015 [US1] Mapear as respostas de erro do controller: `400` (T010), `404` (praça de outro tenant ou sem valor vigente na data) — depende de T014
- [ ] T016 [P] [US1] Criar tela de listagem/cadastro de praças de pedágio em `mile_bag_app/src/features/back-office/pedagios/` (componentes, hook `usePracasPedagio`, `services/pedagiosApi.ts`), incluindo formulário de novo valor de vigência para uma praça existente
- [ ] T017 [US1] Implementar no front-end a exibição do histórico de valores de uma praça (ordenado por vigência) e a mensagem de erro quando a nova vigência é anterior à vigente mais recente (FR-005) — depende de T016

**Checkpoint**: User Story 1 (HU10) completa e testável de forma independente — administrador do tenant consegue cadastrar praças, versionar valores sem perder histórico, e consultar o valor vigente em qualquer data

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T018 [P] Adicionar logging das operações de criação de praça e de novo valor de vigência em `mile_bag_api/src/main/java/com/marillia/milebag/business/pedagio/service/PracaPedagioService.java`
- [ ] T019 Executar a validação de [quickstart.md](./quickstart.md) (cadastro, novo valor sem sobrescrever o anterior, consulta de valor vigente em data passada, cálculo em dobro)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências.
- **Foundational (Phase 2)**: Depende do Setup — BLOQUEIA a User Story 1.
- **User Story 1 (Phase 3)**: Depende da conclusão da Fase 2.
- **Polish (Final Phase)**: Depende da conclusão da Fase 3.

### Dependência entre features

O snapshot de rota (spec-mãe, FR-007) e HU12 (fechamento quinzenal) consomem `PracaPedagioService.valorVigenteEm` (T011) e `calcularReembolso` (T012) — HU10 deve estar implementada antes da implementação de HU12 (ver [tasks.md de HU12](../HU12-gerar-fechamento-quinzenal/tasks.md)). Esta HU (HU10), isoladamente, não depende de nenhuma outra.

### Within User Story 1

- Modelos (T005, T006) e repositórios (T007, T008) antes do serviço (T009–T013).
- `valorVigenteEm` (T011) antes de `calcularReembolso` (T012), que depende dele.
- Serviço completo antes do controller (T014–T015).
- Back-end antes da tela de front-end (T016–T017).

### Parallel Opportunities

- T005, T006, T007, T008 podem rodar em paralelo (arquivos diferentes).
- T016 pode começar em paralelo ao back-end assim que o contrato de API estiver estável, mas T017 depende de T016.

---

## Parallel Example: User Story 1

```bash
# Modelos e repositórios em paralelo:
Task: "Criar entidade PracaPedagio em mile_bag_api/.../model/PracaPedagio.java"
Task: "Criar entidade PracaPedagioValor em mile_bag_api/.../model/PracaPedagioValor.java"
Task: "Criar PracaPedagioRepository em mile_bag_api/.../repository/PracaPedagioRepository.java"
Task: "Criar PracaPedagioValorRepository em mile_bag_api/.../repository/PracaPedagioValorRepository.java"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup.
2. Completar Phase 2: Foundational (schema histórico + RLS) — bloqueante.
3. Completar Phase 3: User Story 1.
4. **Parar e validar**: rodar [quickstart.md](./quickstart.md) de ponta a ponta.
5. Prosseguir para HU12, que depende desta HU estar concluída.

---

## Notes

- `[P]` = arquivos diferentes, sem dependência entre as tarefas marcadas.
- `[US1]` mapeia a tarefa à única User Story desta HU.
- Commit após cada tarefa ou grupo lógico coeso.
- Evitar: tarefas vagas, conflito no mesmo arquivo, dependências que quebrem a testabilidade independente da história.
