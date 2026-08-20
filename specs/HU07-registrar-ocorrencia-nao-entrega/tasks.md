---

description: "Task list for HU07 — Registrar ocorrência de não entrega"
---

# Tasks: Registrar ocorrência de não entrega

**Input**: Design documents from `mile_bag_documentacao/specs/HU07-registrar-ocorrencia-nao-entrega/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/ocorrencia-api.md](./contracts/ocorrencia-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organização**: Esta HU tem uma única User Story (US1, P1). Fases: Setup → Foundational → US1 → Polish.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 (única história desta HU)

## Path Conventions

- Back-end: `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/`
- Front-end (PWA motorista): `mile_bag_app/src/features/motorista/ocorrencia/`

---

## Phase 1: Setup

- [ ] T001 Confirmar/criar o pacote `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/` com as subpastas `controller/`, `service/`, `repository/` e `model/` (conforme `mile_bag_audite/CONVENTIONS.md` §1.1)

---

## Phase 2: Foundational (Blocking Prerequisites)

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T002 Criar migração Flyway para as tabelas `motivo_ocorrencia` (`id`, `tenant_id`, `nome`, `ativo`) e `ocorrencia` (`id`, `entrega_id`, `motivo_ocorrencia_id`, `descricao_adicional`, `data_hora`, `registrado_offline`, coluna de tenant, política de Row Level Security) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 [P] Criar entidade JPA `MotivoOcorrencia` (leitura apenas nesta HU — CRUD completo pertence a HU11) em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/model/MotivoOcorrencia.java`
- [ ] T004 Confirmar que a entidade `Entrega` (de HU02) expõe transição para `EntregaStatus.REABERTA`, preservando o histórico de ocorrências associadas, em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/model/Entrega.java` — se não existir, adicionar
- [ ] T005 Popular ao menos um `MotivoOcorrencia` de exemplo via seed/migração de dados para viabilizar testes locais desta HU antes de HU11 estar implementada (dado de desenvolvimento, não de produção)

**Checkpoint**: Schema, catálogo de motivos e transição de estado da Entrega prontos — implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Registrar ocorrência quando a entrega não se concretiza (Priority: P1) 🎯 MVP

**Goal**: Permitir que o motorista registre uma ocorrência padronizada quando não conseguir concluir uma entrega, reabrindo automaticamente o BDO e preservando o histórico de tentativas — inclusive sem conexão com a internet.

**Independent Test**: Com uma entrega em andamento, registrar uma ocorrência do catálogo do tenant via `POST /api/v1/entregas/{entregaId}/ocorrencias` e verificar que o BDO/entrega é reaberto, sem depender de comprovante ou bagagem carona.

### Implementation for User Story 1

- [ ] T006 [P] [US1] Criar `MotivoOcorrenciaRepository` (leitura, filtrado por tenant e `ativo = true`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/repository/MotivoOcorrenciaRepository.java`
- [ ] T007 [P] [US1] Criar entidade JPA `Ocorrencia` em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/model/Ocorrencia.java`, com os campos de `data-model.md`
- [ ] T008 [P] [US1] Criar `OcorrenciaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/repository/OcorrenciaRepository.java`
- [ ] T009 [US1] Implementar `OcorrenciaService.registrar(entregaId, motivoOcorrenciaId, descricaoAdicional, idLocal)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/service/OcorrenciaService.java`: valida que a entrega não está `ENCERRADA_COM_SUCESSO` (409 se estiver, FR-003), valida que `motivoOcorrenciaId` pertence ao catálogo do mesmo tenant (404 se não), persiste a ocorrência sem alterar registros anteriores (FR-004), transiciona a `Entrega` para `EntregaStatus.REABERTA`, limpando `motoristaResponsavel` (para permitir nova autoatribuição por qualquer motorista — ver `../HU02-autoatribuir-entrega/data-model.md`) e reabre o BDO correspondente (FR-002) — usa `idLocal` como chave de idempotência quando presente (depende de T004, T006, T007, T008)
- [ ] T010 [US1] Implementar `OcorrenciaService.listarPorEntrega(entregaId)` retornando o histórico completo em ordem cronológica (FR-004) (depende de T008)
- [ ] T011 [US1] Implementar endpoint `GET /api/v1/tenants/{tenantId}/motivos-ocorrencia` em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/controller/MotivoOcorrenciaController.java`, conforme `contracts/ocorrencia-api.md` (depende de T006)
- [ ] T012 [US1] Implementar endpoints `POST /api/v1/entregas/{entregaId}/ocorrencias` e `GET /api/v1/entregas/{entregaId}/ocorrencias` em `mile_bag_api/src/main/java/com/marillia/milebag/business/ocorrencia/controller/OcorrenciaController.java`, conforme `contracts/ocorrencia-api.md` (depende de T009, T010)
- [ ] T013 [P] [US1] Criar tela de registro de ocorrência (seleção de motivo a partir do catálogo, campo de descrição opcional) em `mile_bag_app/src/features/motorista/ocorrencia/RegistrarOcorrencia.tsx`, desabilitando a confirmação quando o catálogo estiver vazio (edge case do spec.md)
- [ ] T014 [US1] Integrar as chamadas a `GET /api/v1/tenants/{tenantId}/motivos-ocorrencia` e `POST /api/v1/entregas/{entregaId}/ocorrencias` no client de API em `mile_bag_app/src/features/motorista/ocorrencia/ocorrenciaService.ts`, gerando e enviando `idLocal` (depende de T013)
- [ ] T015 [US1] Garantir que o registro de ocorrência funcione offline: quando sem conexão, usar o catálogo de motivos já sincronizado previamente e salvar o registro na fila local (IndexedDB/Dexie, mesma infraestrutura de HU08) com `registradoOffline = true` em `mile_bag_app/src/features/motorista/offline-sync/` (depende de T014; integra com HU08)
- [ ] T016 [US1] Exibir o histórico de ocorrências de uma entrega (tentativas anteriores) na tela de detalhe da entrega em `mile_bag_app/src/features/motorista/ocorrencia/HistoricoOcorrencias.tsx` (depende de T012)

**Checkpoint**: Neste ponto, a User Story 1 desta HU deve estar completa e testável de forma independente via `quickstart.md`

---

## Phase 4: Polish & Cross-Cutting Concerns

- [ ] T017 [P] Adicionar teste automatizado de isolamento entre tenants para `Ocorrencia` e `MotivoOcorrencia` (RNF01)
- [ ] T018 Run quickstart.md validation — executar o roteiro de [`quickstart.md`](./quickstart.md) de ponta a ponta, incluindo múltiplas ocorrências para a mesma entrega

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: Depende da conclusão do Setup — BLOQUEIA a User Story 1
- **User Story 1 (Phase 3)**: Depende da conclusão da Phase 2
- **Polish (Phase 4)**: Depende da conclusão da Phase 3

### Within User Story 1

- Repositórios e entidades (T006, T007, T008) antes do serviço (T009, T010)
- Serviço antes dos endpoints (T011, T012)
- Tela (T013) pode começar em paralelo ao backend; integração com API (T014) e offline (T015) dependem de T013/T012

### Parallel Opportunities

- T006, T007 e T008 podem rodar em paralelo
- T013 pode rodar em paralelo a T006–T012
- T017 e T018 podem rodar em paralelo

---

## Parallel Example: User Story 1

```bash
# Lançar em paralelo:
Task: "Criar MotivoOcorrenciaRepository em mile_bag_api/.../ocorrencia/MotivoOcorrenciaRepository.java"
Task: "Criar entidade JPA Ocorrencia em mile_bag_api/.../ocorrencia/Ocorrencia.java"
Task: "Criar OcorrenciaRepository em mile_bag_api/.../ocorrencia/OcorrenciaRepository.java"
Task: "Criar tela RegistrarOcorrencia.tsx em mile_bag_app/.../ocorrencia/"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRÍTICO — bloqueia a User Story), incluindo o seed temporário de motivos (T005)
3. Completar Phase 3: User Story 1
4. **PARAR e VALIDAR**: testar a User Story 1 de forma independente via `quickstart.md`
5. Completar Phase 4: Polish

### Notas

- Esta HU depende do catálogo de motivos de ocorrência (CRUD completo em HU11) e integra-se com HU08 (fila offline) — o seed de T005 permite testar esta HU isoladamente antes de HU11 estar pronta.
- Recusa de assinatura pelo recebedor não é comprovante incompleto — é tratada como ocorrência (esta HU), não pela HU06.
- Commitar após cada tarefa ou grupo lógico de tarefas.
- Evitar: tarefas vagas, conflito de mesmo arquivo entre tarefas paralelas, dependências cruzadas que quebrem a testabilidade independente da história.
