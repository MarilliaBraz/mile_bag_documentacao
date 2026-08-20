---

description: "Task list for HU16 — Gerenciar planos de assinatura"
---

# Tasks: HU16 — Gerenciar planos de assinatura

**Input**: Design documents from `mile_bag_documentacao/specs/HU16-gerenciar-planos-assinatura/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/planos-api.md](./contracts/planos-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — omitidos.

**Organization**: HU16 tem uma única User Story (P1). Todas as tarefas de implementação carregam `[US1]`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 desta HU
- Caminhos de arquivo exatos, conforme `plan.md` → Project Structure

---

## Phase 1: Setup

**Purpose**: Preparar a feature de administração de planos no front-end (o back-end reaproveita `business/tenant`, já existente por causa de HU14/HU15).

- [ ] T001 Criar a feature `mile_bag_app/src/features/back-office/planos/` (se ainda não existir)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência de `PlanoAssinatura` e do histórico de plano por tenant — pré-requisito bloqueante de toda a User Story, e também da tarefa T009 de HU14 (associação de tenant a plano).

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase.

- [ ] T002 Criar migração Flyway para a tabela `plano_assinatura` (`id`, `nome`, `limite_motoristas_ativos` > 0, `limite_volume_mensal_bdos` > 0, `criado_em`) em `mile_bag_api/src/main/resources/db/migration/V<próxima>__create_plano_assinatura.sql` — tabela global da plataforma, sem coluna discriminadora de tenant e sem política de Row Level Security por tenant
- [ ] T003 Criar migração Flyway para a tabela `historico_plano_tenant` (`tenant_id` referência, `plano_id` referência, `vigente_desde`, `vigente_ate` nullable, `uso_excedeu_limite_na_troca` booleano) em `mile_bag_api/src/main/resources/db/migration/V<próxima>__create_historico_plano_tenant.sql`, com índice garantindo no máximo um registro com `vigente_ate = null` por `tenant_id` (depende de T002; pode ser a mesma migração ou a seguinte)
- [ ] T004 Criar entidade JPA `PlanoAssinatura` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/model/PlanoAssinatura.java` (depende de T002)
- [ ] T005 Criar entidade JPA `HistoricoPlanoTenant` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/model/HistoricoPlanoTenant.java` (depende de T003)
- [ ] T006 [P] Criar `PlanoAssinaturaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/repository/PlanoAssinaturaRepository.java`, com consulta de contagem de tenants associados a um plano (depende de T004)
- [ ] T007 [P] Criar `HistoricoPlanoTenantRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/repository/HistoricoPlanoTenantRepository.java`, com consulta do registro vigente (`vigenteAte = null`) por `tenantId` (depende de T005)

**Checkpoint**: `PlanoAssinatura` e `HistoricoPlanoTenant` persistíveis e consultáveis — a User Story 1 pode começar. Isso também desbloqueia T009 de HU14 (associação de tenant a plano existente), caso ainda não tenha sido implementada com uma entidade mínima provisória.

---

## Phase 3: User Story 1 - Cadastrar, associar e alterar planos de assinatura (Priority: P1) 🎯 MVP

**Goal**: O administrador da plataforma cadastra planos com limites de motoristas ativos e volume mensal de BDOs, associa tenants a eles e pode trocar o plano de um tenant a qualquer momento, sem reimplantação.

**Independent Test**: Cadastrar um plano com limites definidos, associá-lo a um tenant existente, e depois trocar esse tenant para um segundo plano com limites diferentes — sem qualquer etapa de reimplantação.

### Implementation for User Story 1

- [ ] T008 [P] [US1] Criar `PlanoAssinaturaRequestDTO` (`nome`, `limiteMotoristasAtivos`, `limiteVolumeMensalBdos`) e `PlanoAssinaturaResponseDTO` (inclui contagem de tenants associados) em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/dto/`
- [ ] T009 [US1] Implementar `PlanoAssinaturaService.criar(PlanoAssinaturaRequestDTO)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/PlanoAssinaturaService.java`, validando que ambos os limites são > 0 (depende de T006, T008; FR-001, Edge Case 3)
- [ ] T010 [US1] Implementar `PlanoAssinaturaService.listar()` retornando cada plano com a contagem de tenants associados (via `HistoricoPlanoTenantRepository`, registros com `vigenteAte = null`) (depende de T006, T007)
- [ ] T011 [US1] Implementar `PlanoAssinaturaService.remover(planoId)`, recusando a operação (exceção tratada como HTTP 409) quando houver ao menos um tenant associado (depende de T007; FR-005, Acceptance Scenario 3)
- [ ] T012 [US1] Implementar `PlanoAssinaturaController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/controller/PlanoAssinaturaController.java` com `POST /api/v1/plataforma/planos`, `GET /api/v1/plataforma/planos` e `DELETE /api/v1/plataforma/planos/{id}`, restritos ao administrador da plataforma (depende de T008, T009, T010, T011)
- [ ] T013 [US1] Implementar `TenantPlanoService.trocarPlano(tenantId, planoId)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/TenantPlanoService.java`: em uma transação, fecha o registro `HistoricoPlanoTenant` vigente (`vigenteAte = agora`) e cria um novo (`vigenteDesde = agora`), consultando o uso atual do tenant (motoristas ativos, BDOs do mês) para preencher `usoExcedeuLimiteNaTroca` (depende de T005, T007; FR-002, FR-003, FR-004)
- [ ] T014 [US1] Implementar `PlanoAssinaturaController.trocarPlano(tenantId, planoId)` com `PUT /api/v1/plataforma/tenants/{tenantId}/plano`, retornando `{ planoVigente, usoExcedeuLimiteNaTroca }` e 404 quando `planoId` não existir (depende de T012, T013)
- [ ] T015 [US1] Implementar `PlanoAssinaturaController.consultarPlanoDoTenant()` com `GET /api/v1/plano`, resolvendo `tenantId` a partir do claim do token, retornando `{ planoVigente, usoAtual, dentroDoLimite }` (depende de T007, T012; consumido também por FR-017 do spec-mãe)
- [ ] T016 [US1] Garantir, em `TenantPlanoService.trocarPlano(...)` e na criação inicial via `TenantService.cadastrar(...)` (HU14), que a associação de um tenant a um plano usa exclusivamente `HistoricoPlanoTenant` como fonte de verdade do plano vigente — nenhuma outra tabela deve guardar `planoId` de forma redundante (depende de T013; integra com T009 de HU14, que deve ser ajustada para criar o primeiro registro de `HistoricoPlanoTenant` em vez de um campo direto em `Tenant`, se ainda não estiver assim)
- [ ] T017 [P] [US1] Criar tela de listagem e cadastro de planos em `mile_bag_app/src/features/back-office/planos/PlanosLista.tsx` e `PlanoForm.tsx`, consumindo `GET/POST /api/v1/plataforma/planos`
- [ ] T018 [US1] Adicionar ação de remoção de plano na listagem, com tratamento de erro 409 (orientando a migrar tenants primeiro) em `PlanosLista.tsx` (depende de T017, T011)
- [ ] T019 [US1] Criar tela de troca de plano de um tenant em `mile_bag_app/src/features/back-office/planos/TrocarPlanoTenant.tsx`, consumindo `PUT /api/v1/plataforma/tenants/{tenantId}/plano` e exibindo alerta quando `usoExcedeuLimiteNaTroca = true` (depende de T014; FR-006)

**Checkpoint**: CRUD de planos, associação e troca de plano funcionais de ponta a ponta — testável de forma independente. HU14 depende desta HU para o campo `planoId` do cadastro de tenant.

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T020 [P] Adicionar restrição de acesso: `GET /api/v1/plano` acessível ao administrador do próprio tenant; demais endpoints de `/api/v1/plataforma/planos` restritos ao administrador da plataforma — revisar `PlanoAssinaturaController` para os dois níveis de perfil
- [ ] T021 Revisar consistência de nomenclatura de `PlanoAssinatura`/`HistoricoPlanoTenant` com o uso feito em `TenantService` (HU14) e em qualquer verificação de limite (FR-017 do spec-mãe), já que essas integrações foram desenhadas por HUs diferentes
- [ ] T022 Executar a validação descrita em [quickstart.md](./quickstart.md)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente.
- **Foundational (Phase 2)**: Depende da Phase 1 — BLOQUEIA toda a Phase 3.
- **User Story 1 (Phase 3)**: Depende da Phase 2 completa.
- **Polish (Final Phase)**: Depende da Phase 3 completa.

### Dependência entre features (fora desta HU)

- [HU14](../HU14-cadastrar-empresa-assinante/spec.md) **depende desta HU**: o cadastro de tenant exige um `planoId` válido (FR-002 de HU14). Se HU14 foi implementada antes desta com uma entidade `PlanoAssinatura` mínima provisória, T016 acima cobre a reconciliação — substituir a entidade provisória pela definitiva desta HU.
- [HU15](../HU15-convidar-primeiro-administrador-tenant/spec.md) não depende diretamente desta HU.
- A verificação de limite de plano em tempo real (FR-017 do spec-mãe, ex.: bloquear novo motorista/BDO acima do limite) é responsabilidade das HUs que criam esses recursos, consumindo `GET /api/v1/plano` (T015) — não implementada aqui.

### Parallel Opportunities

- T006 e T007 (Foundational) podem rodar em paralelo.
- T008 e T017 podem rodar em paralelo, mas T017 só é útil de fato após T012 (endpoints) existir.
- T020 e T021 (Polish) podem rodar em paralelo.

---

## Parallel Example: User Story 1

```bash
# Repositórios de PlanoAssinatura e de HistoricoPlanoTenant são arquivos independentes:
Task: "Criar PlanoAssinaturaRepository em mile_bag_api/.../repository/PlanoAssinaturaRepository.java"
Task: "Criar HistoricoPlanoTenantRepository em mile_bag_api/.../repository/HistoricoPlanoTenantRepository.java"
```

---

## Implementation Strategy

### MVP First (User Story 1 única)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRITICAL — inclui a modelagem de histórico de plano)
3. Completar Phase 3: User Story 1
4. **STOP and VALIDATE**: rodar [quickstart.md](./quickstart.md) e confirmar os 4 cenários de aceite do spec
5. Entregar/demonstrar — desbloqueia o campo `planoId` de HU14

### Incremental Delivery

Como esta HU tem uma única User Story, a entrega é binária (T001–T022 completas). Por ser pré-requisito de dado de HU14, recomenda-se implementá-la antes ou em paralelo estreito com HU14, alinhando T016 entre as duas.

---

## Notes

- [P] = arquivos diferentes, sem dependência entre si.
- [US1] mapeia a tarefa à única User Story desta HU.
- Sem tarefas de teste automatizado — não solicitadas no spec.
- Fazer commit após cada tarefa ou grupo lógico coeso.
- Parar no checkpoint da Phase 3 para validar a história de forma independente antes de seguir para o Polish.
