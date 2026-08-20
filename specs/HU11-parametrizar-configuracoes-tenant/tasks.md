---

description: "Task list for HU11 — Parametrizar configurações do tenant"
---

# Tasks: HU11 — Parametrizar configurações do tenant

**Input**: Design documents from `mile_bag_documentacao/specs/HU11-parametrizar-configuracoes-tenant/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/configuracoes-tenant-api.md](./contracts/configuracoes-tenant-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organization**: HU11 tem uma única User Story (P1), que cobre 5 categorias de configuração (motivos de ocorrência, campos de baixa, bases aeroportuárias, companhias atendidas, periodicidade de fechamento). Todas as tarefas de implementação carregam o rótulo `[US1]`.

## Phase 1: Setup

- [ ] T001 Confirmar/criar o subpacote `business/configuracao` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/` (`controller/`, `service/`, `repository/`, `model/`)

## Phase 2: Foundational

**Purpose**: Schema das 5 entidades de configuração e isolamento por tenant, bloqueantes para a User Story

- [ ] T002 Criar migração Flyway para `motivo_ocorrencia` (`id`, `tenant_id`, `descricao`, `ativo`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T003 [P] Criar migração Flyway para `campo_baixa_config` (`id`, `tenant_id`, `campo` [enum], `obrigatorio`) com constraint de unicidade `(tenant_id, campo)` em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T004 [P] Criar migração Flyway para `base_aeroportuaria` (`id`, `tenant_id`, `codigo_iata`, `nome`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T005 [P] Criar migração Flyway para `companhia_aerea_atendida` (`id`, `tenant_id`, `codigo_iata`, `nome`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T006 [P] Criar migração Flyway para `periodicidade_fechamento_config` (`id`, `tenant_id`, `periodicidade` [enum, default `QUINZENAL`], `vigente_desde`) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T007 Adicionar política de Row Level Security filtrando por `tenant_id` nas 5 tabelas de T002–T006, em `mile_bag_api/src/main/resources/db/migration/` — depende de T002–T006

**Checkpoint**: Schema das 5 configurações e isolamento prontos — a implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Parametrizar configurações operacionais do tenant (Priority: P1) 🎯 MVP

**Goal**: Permitir que o administrador do tenant configure, de forma independente das demais empresas, motivos de ocorrência, campos obrigatórios do formulário de baixa, bases aeroportuárias, companhias aéreas atendidas e periodicidade do fechamento.

**Independent Test**: Alterar cada uma das cinco configurações para um tenant já operacional e verificar que (a) a alteração é refletida onde a configuração é consumida e (b) nenhum outro tenant é afetado.

### Implementation for User Story 1 — Motivos de ocorrência

- [ ] T008 [P] [US1] Criar entidade `MotivoOcorrencia` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/MotivoOcorrencia.java` (`id`, `tenantId`, `descricao`, `ativo`)
- [ ] T009 [P] [US1] Criar `MotivoOcorrenciaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/MotivoOcorrenciaRepository.java` com busca por `tenantId` e por `tenantId + ativo=true`
- [ ] T010 [US1] Implementar `MotivoOcorrenciaService` (criar, listar ativos, desativar via `PATCH` — nunca excluir, FR-009) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/service/MotivoOcorrenciaService.java` — depende de T008, T009

### Implementation for User Story 1 — Campos obrigatórios do formulário de baixa

- [ ] T011 [P] [US1] Criar entidade `CampoBaixaConfig` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/CampoBaixaConfig.java` (`id`, `tenantId`, `campo` como enum `CampoBaixa`, `obrigatorio`)
- [ ] T012 [P] [US1] Criar `CampoBaixaConfigRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/CampoBaixaConfigRepository.java` com busca por `tenantId`
- [ ] T013 [US1] Implementar `CampoBaixaConfigService.atualizar(tenantId, List<CampoObrigatorio>)`, validando que cada `campo` pertence ao catálogo fixo `CampoBaixa` (`400` se não) — depende de T011, T012

### Implementation for User Story 1 — Bases aeroportuárias e companhias atendidas

- [ ] T014 [P] [US1] Criar entidade `BaseAeroportuaria` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/BaseAeroportuaria.java` (`id`, `tenantId`, `codigoIata`, `nome`)
- [ ] T015 [P] [US1] Criar entidade `CompanhiaAereaAtendida` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/CompanhiaAereaAtendida.java` (`id`, `tenantId`, `codigoIata`, `nome`)
- [ ] T016 [P] [US1] Criar `BaseAeroportuariaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/BaseAeroportuariaRepository.java`
- [ ] T017 [P] [US1] Criar `CompanhiaAereaAtendidaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/CompanhiaAereaAtendidaRepository.java`
- [ ] T018 [US1] Implementar `BaseAeroportuariaService` (criar, listar, remover) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/service/BaseAeroportuariaService.java` — depende de T014, T016
- [ ] T019 [US1] Implementar `CompanhiaAereaAtendidaService` (criar, listar, remover) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/service/CompanhiaAereaAtendidaService.java` — depende de T015, T017

### Implementation for User Story 1 — Periodicidade de fechamento

- [ ] T020 [P] [US1] Criar entidade `PeriodicidadeFechamentoConfig` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/model/PeriodicidadeFechamentoConfig.java` (`id`, `tenantId`, `periodicidade` enum default `QUINZENAL`, `vigenteDesde`)
- [ ] T021 [P] [US1] Criar `PeriodicidadeFechamentoConfigRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/repository/PeriodicidadeFechamentoConfigRepository.java`
- [ ] T022 [US1] Implementar `PeriodicidadeFechamentoConfigService.agendarNovaPeriodicidade(tenantId, periodicidade, vigenteDesde)`, rejeitando (`400`) quando `vigenteDesde` cair dentro do período de fechamento já em andamento (FR-008) — depende de T020, T021
- [ ] T023 [US1] Implementar `PeriodicidadeFechamentoConfigService.periodicidadeVigenteEm(tenantId, data)` (mesmo padrão de resolução por vigência de [HU10](../HU10-cadastrar-pracas-pedagio/tasks.md)) — depende de T021

### Implementation for User Story 1 — API e front-end

- [ ] T024 [US1] Criar `ConfiguracaoTenantController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/controller/ConfiguracaoTenantController.java` implementando todos os endpoints de [contracts/configuracoes-tenant-api.md](./contracts/configuracoes-tenant-api.md) (motivos, campos-baixa, bases-aeroportuarias, companhias-aereas, periodicidade-fechamento) — depende de T010, T013, T018, T019, T022, T023
- [ ] T025 [US1] Garantir, no `ConfiguracaoTenantController`, que os endpoints de leitura (`GET`) das 5 categorias são acessíveis também ao perfil motorista, restrito ao próprio tenant (RNF01/FR-010) — depende de T024
- [ ] T026 [P] [US1] Criar tela de configurações do tenant em `mile_bag_app/src/features/back-office/configuracao-tenant/` com 5 seções (motivos de ocorrência, campos de baixa, bases aeroportuárias, companhias atendidas, periodicidade), `services/configuracoesTenantApi.ts` consumindo os endpoints do contrato — depende de T024

**Checkpoint**: User Story 1 (HU11) completa e testável de forma independente — administrador do tenant consegue configurar as 5 categorias, com isolamento entre tenants garantido e sem impacto em fechamentos/ocorrências já registrados

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T027 [P] Adicionar logging das alterações de configuração (todas as 5 categorias) em `mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/service/`
- [ ] T028 Executar a validação de [quickstart.md](./quickstart.md) (alterar cada uma das 5 configurações e confirmar reflexo/isolamento)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências.
- **Foundational (Phase 2)**: Depende do Setup — BLOQUEIA a User Story 1.
- **User Story 1 (Phase 3)**: Depende da conclusão da Fase 2.
- **Polish (Final Phase)**: Depende da conclusão da Fase 3.

### Dependência entre features

O catálogo de `MotivoOcorrencia` (T008–T010) é consumido pelo domínio `ocorrencia` (HU07, fora desta pasta); os `CampoBaixaConfig` (T011–T013) são consumidos pela interface de baixa do motorista (HU06/HU07); a `PeriodicidadeFechamentoConfig` (T020–T023) é consumida por HU12 (fechamento quinzenal). Esta HU deve estar implementada antes dessas três, mas não depende de nenhuma delas.

### Within User Story 1

- As 5 sub-áreas (motivos, campos, bases/companhias, periodicidade) são independentes entre si — modelos e repositórios de cada uma (T008–T009, T011–T012, T014–T017, T020–T021) podem ser feitos em paralelo.
- Dentro de cada sub-área, modelo/repositório antes do service; todos os services antes do controller único (T024).
- Controller antes da tela de front-end (T026).

### Parallel Opportunities

- T003–T006 (migrações das 5 tabelas) podem rodar em paralelo.
- T008, T009, T011, T012, T014–T017, T020, T021 podem todos rodar em paralelo entre si (entidades e repositórios de sub-áreas distintas, arquivos diferentes).

---

## Parallel Example: User Story 1

```bash
# Modelos e repositórios das 5 sub-áreas em paralelo:
Task: "Criar entidade MotivoOcorrencia em mile_bag_api/.../model/MotivoOcorrencia.java"
Task: "Criar entidade CampoBaixaConfig em mile_bag_api/.../model/CampoBaixaConfig.java"
Task: "Criar entidade BaseAeroportuaria em mile_bag_api/.../model/BaseAeroportuaria.java"
Task: "Criar entidade CompanhiaAereaAtendida em mile_bag_api/.../model/CompanhiaAereaAtendida.java"
Task: "Criar entidade PeriodicidadeFechamentoConfig em mile_bag_api/.../model/PeriodicidadeFechamentoConfig.java"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup.
2. Completar Phase 2: Foundational (schema das 5 tabelas + RLS) — bloqueante.
3. Completar Phase 3: User Story 1 (as 5 sub-áreas podem ser paralelizadas entre desenvolvedores).
4. **Parar e validar**: rodar [quickstart.md](./quickstart.md) de ponta a ponta.
5. Disponibilizar para consumo por HU06/HU07 (campos de baixa, motivos) e HU12 (periodicidade).

---

## Notes

- `[P]` = arquivos diferentes, sem dependência entre as tarefas marcadas.
- `[US1]` mapeia a tarefa à única User Story desta HU.
- Commit após cada tarefa ou grupo lógico coeso.
- Evitar: tarefas vagas, conflito no mesmo arquivo, dependências que quebrem a testabilidade independente da história.
