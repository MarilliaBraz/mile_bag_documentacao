---

description: "Task list for HU06 — Registrar o comprovante de entrega"
---

# Tasks: Registrar o comprovante de entrega

**Input**: Design documents from `mile_bag_documentacao/specs/HU06-registrar-comprovante-entrega/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/comprovante-entrega-api.md](./contracts/comprovante-entrega-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organização**: Esta HU tem uma única User Story (US1, P1). Fases: Setup → Foundational → US1 → Polish.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 (única história desta HU)

## Path Conventions

- Back-end: `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/`
- Front-end (PWA motorista): `mile_bag_app/src/features/motorista/comprovante-entrega/`

---

## Phase 1: Setup

- [ ] T001 Confirmar/criar o pacote `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/` com as subpastas `controller/`, `service/`, `repository/` e `model/` (domínio próprio, separado de `entrega`, conforme `plan.md` e `mile_bag_audite/CONVENTIONS.md` §1.1)

---

## Phase 2: Foundational (Blocking Prerequisites)

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T002 Criar migração Flyway para a tabela `comprovante_entrega` (colunas: `id`, `entrega_id` [único], `foto_documento_url`, `identificacao_recebedor`, `data_hora`, `latitude`, `longitude`, `registrado_offline`, coluna de tenant, política de Row Level Security) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 Confirmar que a entidade `Entrega` (de HU02) expõe o campo `status` (`EntregaStatus.ENCERRADA_COM_SUCESSO` — ver `../HU02-autoatribuir-entrega/data-model.md`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/entrega/model/Entrega.java` — se não existir, adicionar
- [ ] T004 Configurar o repositório de armazenamento de imagens por tenant (interface já usada por HU01 para a foto do BDO) para aceitar também a foto do comprovante, em `mile_bag_api/src/main/java/com/marillia/milebag/core/storage/`

**Checkpoint**: Schema, transição de estado da Entrega e armazenamento de imagem prontos — implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Registrar o comprovante de entrega ao concluir a devolução (Priority: P1) 🎯 MVP

**Goal**: Permitir que o motorista registre, ao concluir a entrega, o comprovante completo (foto do documento assinado, identificação do recebedor, data, hora e geolocalização), encerrando a entrega com sucesso — inclusive sem conexão com a internet.

**Independent Test**: Com uma entrega em andamento (viagem já iniciada, HU04), registrar o comprovante via `POST /api/v1/entregas/{entregaId}/comprovante` e verificar que a entrega muda para `ENCERRADA_COM_SUCESSO`, sem depender de bagagem carona ou de ocorrência.

### Implementation for User Story 1

- [ ] T005 [P] [US1] Criar entidade JPA `ComprovanteEntrega` em `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/model/ComprovanteEntrega.java`, com os campos de `data-model.md`
- [ ] T006 [P] [US1] Criar `ComprovanteEntregaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/repository/ComprovanteEntregaRepository.java`
- [ ] T007 [US1] Implementar `ComprovanteEntregaService.registrar(entregaId, foto, identificacaoRecebedor, dataHora, latitude, longitude, idLocal)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/service/ComprovanteEntregaService.java`: valida presença de todos os campos obrigatórios (422 se ausente, FR-002), rejeita se já existir comprovante para a entrega (409, FR-004), salva a imagem via repositório de armazenamento (T004), persiste o registro e transiciona a `Entrega` para `ENCERRADA_COM_SUCESSO` (FR-003) — usa `idLocal` como chave de idempotência quando presente (depende de T003, T004, T005, T006)
- [ ] T008 [US1] Implementar endpoint `POST /api/v1/entregas/{entregaId}/comprovante` (multipart/form-data) em `mile_bag_api/src/main/java/com/marillia/milebag/business/comprovante/controller/ComprovanteEntregaController.java`, conforme `contracts/comprovante-entrega-api.md` (depende de T007)
- [ ] T009 [P] [US1] Criar tela de captura do comprovante (foto do documento, campo de recebedor, captura de data/hora e geolocalização automáticas) em `mile_bag_app/src/features/motorista/comprovante-entrega/RegistrarComprovante.tsx`
- [ ] T010 [US1] Tratar falha de captura de geolocalização na UI: reter os demais campos preenchidos e permitir nova tentativa de captura de coordenadas sem perder o restante do formulário (edge case do spec.md) em `mile_bag_app/src/features/motorista/comprovante-entrega/RegistrarComprovante.tsx` (depende de T009)
- [ ] T011 [US1] Integrar a chamada ao `POST /api/v1/entregas/{entregaId}/comprovante` no client de API em `mile_bag_app/src/features/motorista/comprovante-entrega/comprovanteService.ts`, gerando e enviando `idLocal` (depende de T009)
- [ ] T012 [US1] Garantir que o registro do comprovante funcione offline: quando sem conexão, salvar o registro (incluindo a foto) na fila local (IndexedDB/Dexie, mesma infraestrutura de HU08) com `registradoOffline = true`, sincronizando depois em `mile_bag_app/src/features/motorista/offline-sync/` (depende de T011; integra com HU08)
- [ ] T013 [US1] Adicionar tratamento de erro na UI para os casos 422/409/404 (campo ausente, comprovante já existente, entrega inexistente) em `mile_bag_app/src/features/motorista/comprovante-entrega/RegistrarComprovante.tsx` (depende de T011)

**Checkpoint**: Neste ponto, a User Story 1 desta HU deve estar completa e testável de forma independente via `quickstart.md`

---

## Phase 4: Polish & Cross-Cutting Concerns

- [ ] T014 [P] Adicionar teste automatizado de isolamento entre tenants para `ComprovanteEntrega` (RNF01)
- [ ] T015 [P] Confirmar tratamento de dados pessoais do recebedor (nome) e geolocalização conforme LGPD (retenção/acesso restrito por perfil), alinhado ao Constitution Check do `plan.md`
- [ ] T016 Run quickstart.md validation — executar o roteiro de [`quickstart.md`](./quickstart.md) de ponta a ponta, incluindo o cenário offline

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: Depende da conclusão do Setup — BLOQUEIA a User Story 1
- **User Story 1 (Phase 3)**: Depende da conclusão da Phase 2
- **Polish (Phase 4)**: Depende da conclusão da Phase 3

### Within User Story 1

- Entidade e repositório (T005, T006) antes do serviço (T007)
- Serviço (T007) antes do endpoint (T008)
- Tela de captura (T009) pode começar em paralelo ao backend; tratamento de erro/geolocalização (T010) e integração com API (T011) dependem de T009
- Integração offline (T012) depende de T011

### Parallel Opportunities

- T005 e T006 podem rodar em paralelo
- T009 (tela) pode rodar em paralelo a T005–T008 (backend)
- T014 e T015 podem rodar em paralelo entre si e com T016

---

## Parallel Example: User Story 1

```bash
# Lançar em paralelo:
Task: "Criar entidade JPA ComprovanteEntrega em mile_bag_api/.../comprovante/ComprovanteEntrega.java"
Task: "Criar ComprovanteEntregaRepository em mile_bag_api/.../comprovante/ComprovanteEntregaRepository.java"
Task: "Criar tela RegistrarComprovante.tsx em mile_bag_app/.../comprovante-entrega/"
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

- Esta HU depende de uma entrega já em andamento (HU04) e integra-se com HU08 (fila offline) — mas é implementável e testável de forma independente, desde que essa dependência já exista no ambiente de teste.
- O recebedor que se recusa a assinar não é tratado aqui — vira ocorrência (HU07), fora do escopo desta HU.
- Commitar após cada tarefa ou grupo lógico de tarefas.
- Evitar: tarefas vagas, conflito de mesmo arquivo entre tarefas paralelas, dependências cruzadas que quebrem a testabilidade independente da história.
