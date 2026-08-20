---

description: "Task list for HU13 — Emitir relatórios com identidade visual do tenant"
---

# Tasks: HU13 — Emitir relatórios com identidade visual do tenant

**Input**: Design documents from `mile_bag_documentacao/specs/HU13-emitir-relatorios-identidade-visual/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/identidade-visual-api.md](./contracts/identidade-visual-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — omitidos.

**Organization**: HU13 tem uma única User Story (P1). Todas as tarefas de implementação carregam `[US1]`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 desta HU
- Caminhos de arquivo exatos, conforme `plan.md` → Project Structure

---

## Phase 1: Setup

**Purpose**: Preparar o pacote de configuração de tenant para receber a identidade visual (reaproveita `business/configuracao`, já existente por causa de HU11 — se ainda não existir no seu ambiente, criar a estrutura base primeiro).

- [ ] T001 Garantir dependência do driver de upload multipart no `mile_bag_api/pom.xml` (Spring Web já inclui suporte a `MultipartFile`; confirmar `spring-boot-starter-web` presente)
- [ ] T002 [P] Confirmar/loga a dependência de geração de PDF (JasperReports Library, conforme `plan.md` → Primary Dependencies) no `mile_bag_api/pom.xml`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Estrutura de persistência e de armazenamento de arquivo que a User Story depende.

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase.

- [ ] T003 Criar migração Flyway para a tabela `identidade_visual_tenant` (`tenant_id` referência única, `logotipo_path`, `cabecalho`, `atualizado_em`) em `mile_bag_api/src/main/resources/db/migration/V<próxima>__create_identidade_visual_tenant.sql`, com política de Row Level Security por `tenant_id`
- [ ] T004 Criar entidade JPA `IdentidadeVisualTenant` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/IdentidadeVisualTenant.java`, mapeando os campos de T003
- [ ] T005 Criar `IdentidadeVisualTenantRepository` (Spring Data JPA) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/IdentidadeVisualTenantRepository.java`
- [ ] T006 Confirmar/instanciar a interface de repositório de arquivos por tenant (volume em disco com prefixo por tenant, ver plano-mãe → Storage) em `mile_bag_api/src/main/java/com/marillia/milebag/core/storage/TenantFileStorage.java`, reaproveitada por T009

**Checkpoint**: Estrutura de dados e armazenamento prontas — a User Story 1 pode começar.

---

## Phase 3: User Story 1 - Emitir relatórios com identidade visual do tenant (Priority: P1) 🎯 MVP

**Goal**: O administrador do tenant configura logotipo e cabeçalho; todo relatório de fechamento gerado a partir daí sai automaticamente com essa identidade, sem retrabalho manual.

**Independent Test**: Configurar a identidade visual de um tenant já operacional (com fechamentos existentes de HU12) e verificar que o próximo relatório emitido reflete essa identidade, sem alterar os valores financeiros do fechamento.

### Implementation for User Story 1

- [ ] T007 [P] [US1] Criar `IdentidadeVisualTenantDTO` (request/response) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/dto/IdentidadeVisualTenantDTO.java`, espelhando o formato do contrato (`logotipoUrl`, `cabecalho`, `atualizadoEm`)
- [ ] T008 [US1] Implementar `IdentidadeVisualTenantService` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/service/IdentidadeVisualTenantService.java` com métodos `buscar(tenantId)` e `salvar(tenantId, logotipo, cabecalho)` (depende de T004, T005)
- [ ] T009 [US1] Implementar upload/substituição do arquivo de logotipo no `TenantFileStorage` (prefixo por tenant) dentro de `IdentidadeVisualTenantService.salvar(...)`, validando formato e tamanho antes de persistir (depende de T006, T008)
- [ ] T010 [US1] Implementar `IdentidadeVisualTenantController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/controller/IdentidadeVisualTenantController.java` com `GET /api/v1/identidade-visual` e `PUT /api/v1/identidade-visual` (multipart/form-data), resolvendo `tenantId` a partir do claim do token — nunca de parâmetro de entrada (depende de T007, T008)
- [ ] T011 [US1] Adicionar validação e resposta 400 para logotipo em formato/tamanho não suportado, preservando a identidade anterior, em `IdentidadeVisualTenantService.salvar(...)` (depende de T009)
- [ ] T012 [US1] Adicionar resposta 403 quando o usuário autenticado não for administrador do tenant, em `IdentidadeVisualTenantController` (depende de T010)
- [ ] T013 [US1] Integrar leitura de `IdentidadeVisualTenantService.buscar(tenantId)` na geração do PDF de fechamento, em `mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/service/RelatorioFechamentoService.java` (definido em HU12), aplicando logotipo/cabeçalho ao template JasperReports/PDFBox quando presentes e aparência padrão neutra quando ausentes (depende de T008; integra com HU12)
- [ ] T014 [US1] Garantir, no ponto de leitura de T013, que a identidade lida é sempre a vigente no momento da geração (nunca retroativa a relatórios já emitidos) — sem necessidade de campo extra, apenas de não cachear/persistir a identidade dentro do registro de fechamento
- [ ] T015 [P] [US1] Criar tela de upload/edição de logotipo e cabeçalho em `mile_bag_app/src/features/back-office/configuracao-tenant/IdentidadeVisualForm.tsx`, consumindo `GET`/`PUT /api/v1/identidade-visual`
- [ ] T016 [US1] Exibir preview da identidade visual aplicada ao lado do botão de download do relatório em `mile_bag_app/src/features/back-office/fechamento/FechamentoRelatorioCard.tsx` (depende de T015; feature `fechamento` de HU12)
- [ ] T017 [US1] Adicionar tratamento de erro na UI para resposta 400 (formato/tamanho de logotipo inválido) em `IdentidadeVisualForm.tsx` (depende de T015, T011)

**Checkpoint**: Identidade visual configurável por tenant e aplicada automaticamente nos relatórios de fechamento — testável de ponta a ponta de forma independente.

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T018 [P] Adicionar índice único em `tenant_id` na tabela `identidade_visual_tenant` (reforço da regra "no máximo uma identidade vigente por tenant", RI07) na migração de T003 ou em migração complementar
- [ ] T019 Revisar logs/erros de upload de logotipo para não vazar caminho de arquivo de outro tenant nas mensagens de erro
- [ ] T020 Executar a validação descrita em [quickstart.md](./quickstart.md)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente.
- **Foundational (Phase 2)**: Depende da Phase 1 — BLOQUEIA toda a Phase 3.
- **User Story 1 (Phase 3)**: Depende da Phase 2 completa.
- **Polish (Final Phase)**: Depende da Phase 3 completa.

### Dependência entre features (fora desta HU)

Esta HU consome a entidade e o endpoint de relatório de fechamento definidos em [HU12](../HU12-gerar-fechamento-quinzenal/spec.md). T013 assume que `RelatorioFechamentoService` (HU12) já existe — se a implementação de HU12 ainda não estiver concluída, T013 fica bloqueada até lá.

### Parallel Opportunities

- T001 e T002 (Setup) podem rodar em paralelo.
- T007 e T015 podem rodar em paralelo (back-end DTO vs. tela de front-end), mas T015 só é útil de fato após T010 (endpoint) existir.
- T018 (Polish) pode rodar em paralelo com T019.

---

## Parallel Example: User Story 1

```bash
# Back-end (DTO) e front-end (tela) podem começar em paralelo, já que dependem de arquivos diferentes:
Task: "Criar IdentidadeVisualTenantDTO em mile_bag_api/.../dto/IdentidadeVisualTenantDTO.java"
Task: "Criar tela de upload/edição em mile_bag_app/src/features/back-office/configuracao-tenant/IdentidadeVisualForm.tsx"
```

---

## Implementation Strategy

### MVP First (User Story 1 única)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRITICAL — bloqueia a User Story)
3. Completar Phase 3: User Story 1
4. **STOP and VALIDATE**: rodar [quickstart.md](./quickstart.md) e confirmar os 4 cenários de aceite do spec
5. Entregar/demonstrar

### Incremental Delivery

Como esta HU tem uma única User Story, a entrega é binária: completa (todas as tarefas T001–T020) ou não entregue. Não há fatias adicionais dentro desta feature — a próxima fatia de valor é a HU seguinte no roadmap (ex. HU16, gerenciamento de planos).

---

## Notes

- [P] = arquivos diferentes, sem dependência entre si.
- [US1] mapeia a tarefa à única User Story desta HU.
- Sem tarefas de teste automatizado — não solicitadas no spec.
- Fazer commit após cada tarefa ou grupo lógico coeso.
- Parar no checkpoint da Phase 3 para validar a história de forma independente antes de seguir para o Polish.
