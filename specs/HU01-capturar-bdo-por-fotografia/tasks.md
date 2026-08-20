---

description: "Task list for HU01 — Capturar o BDO por fotografia"
---

# Tasks: HU01 — Capturar o BDO por fotografia

**Input**: Design documents from `mile_bag_documentacao/specs/HU01-capturar-bdo-por-fotografia/`

**Prerequisites**: plan.md, spec.md, data-model.md, contracts/bdo-api.md, research.md, quickstart.md

**Tests**: Não solicitados no spec desta HU — nenhuma tarefa de teste incluída.

**Organization**: Esta HU tem uma única User Story (US1, P1) — todas as tarefas de implementação carregam o rótulo `[US1]`.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência de tarefa incompleta)
- **[US1]**: Tarefa da User Story 1 (única história desta HU)
- Caminhos de arquivo exatos incluídos em cada descrição

---

## Phase 1: Setup

**Purpose**: Estrutura inicial de pacotes/feature para esta HU

- [ ] T001 Criar estrutura de pacotes `controller/`, `service/`, `repository/`, `model/`, `mapper/` em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/`
- [ ] T002 [P] Criar estrutura de pastas `components/`, `hooks/`, `services/`, `types/` em `mile_bag_app/src/features/motorista/captura-bdo/`
- [ ] T003 [P] Adicionar dependência Tess4J (Tesseract OCR 5) em `mile_bag_api/pom.xml`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência e isolamento multi-tenant que bloqueiam a implementação da história

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T004 Criar migração Flyway para a tabela `bdo` (`id`, `tenant_id`, `imagem_original`, `campos_extraidos` — JSON, `status_conferencia`, `data_captura`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T005 [P] Criar entidade `Bdo` em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/model/Bdo.java`, campos conforme `data-model.md`
- [ ] T006 [P] Configurar interface de repositório para armazenamento da imagem original (volume em disco com prefixo por tenant) em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/repository/ImagemBdoRepository.java`
- [ ] T007 Adicionar política de Row Level Security para isolamento por tenant na tabela `bdo`, na mesma migração de T004

**Checkpoint**: Fundação pronta — a User Story 1 pode ser implementada

---

## Phase 3: User Story 1 - Capturar o BDO por fotografia (Priority: P1) 🎯 MVP

**Goal**: O motorista fotografa o BDO, o sistema extrai automaticamente os campos do padrão WorldTracer e exige confirmação em tela de conferência antes de gravar o registro, preservando a imagem original como prova documental.

**Independent Test**: Fotografar um BDO real e verificar que os campos extraídos aparecem corretamente na tela de conferência, sem depender de nenhuma outra HU.

### Implementation for User Story 1

- [ ] T008 [P] [US1] Implementar `BdoRepository` (persistência da entidade `Bdo`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/repository/BdoRepository.java`
- [ ] T009 [US1] Implementar `BdoOcrService`, integrando Tess4J para extrair os campos do padrão WorldTracer a partir da imagem capturada, em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/service/BdoOcrService.java` (depende de T005)
- [ ] T010 [US1] Implementar `BdoService`, orquestrando upload da imagem, chamada ao OCR e persistência com `statusConferencia = pendente`, em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/service/BdoService.java` (depende de T008, T009)
- [ ] T011 [US1] Implementar endpoint `POST /api/v1/bdos` (upload multipart + extração) em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/controller/BdoController.java` (depende de T010)
- [ ] T012 [US1] Implementar endpoint `PATCH /api/v1/bdos/{id}/conferencia`, validando que o registro só grava com `statusConferencia = confirmado` mediante confirmação explícita (FR-003, FR-005), em `BdoController.java` (depende de T010)
- [ ] T013 [US1] Implementar endpoint `GET /api/v1/bdos?possivelDuplicidade=true&campoChave={valor}` para sinalização de duplicidade, em `BdoController.java` (depende de T010)
- [ ] T014 [US1] Adicionar validação de campos obrigatórios ausentes na confirmação, retornando `400` (FR-005, contrato de erro), em `BdoService.java`
- [ ] T015 [P] [US1] Implementar componente de captura de câmera em `mile_bag_app/src/features/motorista/captura-bdo/components/CapturaBdo.tsx`
- [ ] T016 [P] [US1] Implementar tela de conferência com campos extraídos editáveis em `mile_bag_app/src/features/motorista/captura-bdo/components/TelaConferenciaBdo.tsx`
- [ ] T017 [US1] Implementar chamadas de API (upload, conferência, checagem de duplicidade) em `mile_bag_app/src/features/motorista/captura-bdo/services/bdoService.ts` (depende de T011, T012, T013 — contrato de API)
- [ ] T018 [US1] Conectar o fluxo captura → conferência → confirmação, incluindo aviso de duplicidade (Edge Case), em `mile_bag_app/src/features/motorista/captura-bdo/hooks/useCapturaBdo.ts` (depende de T015, T016, T017)
- [ ] T019 [US1] Adicionar tratamento de foto ilegível com opção de refazer, antes da extração, em `CapturaBdo.tsx` (Edge Case do spec.md)

**Checkpoint**: A User Story 1 está funcional e testável de forma independente

---

## Phase Final: Polish & Cross-Cutting Concerns

- [ ] T020 [P] Adicionar logging estruturado das operações de captura e confirmação de BDO em `mile_bag_api/src/main/java/com/marillia/milebag/business/bdo/service/BdoService.java`
- [ ] T021 Executar a validação de `quickstart.md` desta HU (`mile_bag_documentacao/specs/HU01-capturar-bdo-por-fotografia/quickstart.md`)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: depende da conclusão do Setup — bloqueia a User Story 1
- **User Story 1 (Phase 3)**: depende da conclusão da Phase 2
- **Polish (Final Phase)**: depende da conclusão da User Story 1

### Within User Story 1

- Modelos/repositório (T008) antes de serviços (T009, T010)
- Serviços antes de endpoints (T011–T013)
- Componentes de UI (T015, T016) podem ser feitos em paralelo aos endpoints de back-end, mas a integração (T017, T018) depende do contrato de API já implementado

### Parallel Opportunities

- T002 e T003 (Setup) podem rodar em paralelo com T001
- T005 e T006 (Foundational) podem rodar em paralelo entre si
- T008 pode rodar em paralelo com T009 (ambos dependem só de T005)
- T015 e T016 (componentes de UI) podem rodar em paralelo entre si

---

## Parallel Example: User Story 1

```bash
# Repositório e OCR podem ser implementados em paralelo (dependem apenas do modelo, T005):
Task: "Implementar BdoRepository em mile_bag_api/.../business/bdo/repository/BdoRepository.java"
Task: "Implementar BdoOcrService em mile_bag_api/.../business/bdo/service/BdoOcrService.java"

# Componentes de front-end em paralelo:
Task: "Implementar CapturaBdo.tsx em mile_bag_app/.../captura-bdo/components/"
Task: "Implementar TelaConferenciaBdo.tsx em mile_bag_app/.../captura-bdo/components/"
```

---

## Implementation Strategy

### MVP (única User Story desta HU)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (bloqueia a história)
3. Completar Phase 3: User Story 1
4. **Parar e validar**: rodar `quickstart.md` desta HU de forma isolada
5. Entregar/demonstrar — esta HU é, por si só, um incremento útil (registro digital do BDO substitui a digitação manual)

### Observação

Esta HU é a base de toda a cadeia de entrega: HU02 (autoatribuição) depende de um BDO confirmado por esta história. Concluir e validar esta HU antes de iniciar HU02.

---

## Notes

- `[P]` = arquivos diferentes, sem dependências entre si
- `[US1]` mapeia a tarefa à única história desta HU, para rastreabilidade
- Sem tarefas de teste (não solicitadas no spec desta HU)
- Fazer commit após cada tarefa ou grupo lógico de tarefas
- Parar no checkpoint da Phase 3 para validar a história isoladamente antes de seguir para HU02
