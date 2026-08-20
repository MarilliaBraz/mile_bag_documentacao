---

description: "Task list for HU14 — Cadastrar empresa assinante (tenant)"
---

# Tasks: HU14 — Cadastrar empresa assinante (tenant)

**Input**: Design documents from `mile_bag_documentacao/specs/HU14-cadastrar-empresa-assinante/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/tenants-api.md](./contracts/tenants-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — omitidos.

**Organization**: HU14 tem uma única User Story (P1). Todas as tarefas de implementação carregam `[US1]`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 desta HU
- Caminhos de arquivo exatos, conforme `plan.md` → Project Structure

---

## Phase 1: Setup

**Purpose**: Preparar o subpacote `business/tenant`, compartilhado entre HU14, HU15 e HU16.

- [ ] T001 Criar a estrutura de pastas `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/{controller,service,repository,model,dto}` (se ainda não existir)
- [ ] T002 [P] Criar a feature `mile_bag_app/src/features/back-office/tenant-onboarding/` (se ainda não existir)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência do Tenant, isolamento por Row Level Security e a política de RLS genérica do schema — pré-requisito bloqueante de toda entidade multi-tenant do sistema, não só desta HU.

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase.

- [ ] T003 Criar migração Flyway para a tabela `tenant` (`id`, `identificador_empresa` único, `nome`, `plano_id` referência, `criado_em`, `status`) em `mile_bag_api/src/main/resources/db/migration/V<próxima>__create_tenant.sql`
- [ ] T004 Na mesma migração de T003 (ou em migração complementar imediatamente seguinte), habilitar Row Level Security na tabela `tenant` e criar a política que filtra por `id` a partir do claim de tenant do JWT — esta é a política-base referenciada pelo plano-mãe ("Isolamento multi-tenant"); tabelas de domínio futuras (motoristas, BDOs etc.) reutilizam o mesmo padrão de coluna discriminadora + política
- [ ] T005 Criar entidade JPA `Tenant` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/model/Tenant.java`, mapeando os campos de T003 (depende de T003)
- [ ] T006 Criar `TenantRepository` (Spring Data JPA) em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/repository/TenantRepository.java` (depende de T005)
- [ ] T007 Garantir que o filtro de tenant do interceptador/security config (`core/config`, previsto no plano-mãe) reconhece a tabela `tenant` como caso especial — a própria linha do tenant é lida antes de o claim existir (ex.: leitura administrativa via perfil "administrador da plataforma", que não carrega claim de tenant)

**Checkpoint**: Tabela `tenant` criada, isolada por RLS e acessível via repositório — a User Story 1 pode começar.

---

## Phase 3: User Story 1 - Cadastrar uma nova empresa assinante (Priority: P1) 🎯 MVP

**Goal**: O administrador da plataforma cadastra uma nova empresa como tenant, associada a um plano de assinatura existente, sem qualquer etapa de implantação técnica dedicada, com isolamento de dados garantido desde o primeiro registro.

**Independent Test**: Cadastrar uma nova empresa com um plano associado e verificar que o registro do tenant existe, está isolado de outros tenants já cadastrados, e não deixa pendência de configuração de infraestrutura em aberto.

### Implementation for User Story 1

- [ ] T008 [P] [US1] Criar `CadastroTenantRequestDTO` (`identificadorEmpresa`, `nome`, `planoId`) e `TenantResponseDTO` (`id`, `identificadorEmpresa`, `nome`, `planoId`, `status`, `criadoEm`) em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/dto/`
- [ ] T009 [US1] Implementar `TenantService.cadastrar(CadastroTenantRequestDTO)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/TenantService.java`, validando que `planoId` existe e está ativo antes de persistir (depende de T006, T008; integra com o `PlanoAssinaturaRepository` de HU16)
- [ ] T010 [US1] Envolver a criação do `Tenant` em `TenantService.cadastrar(...)` em uma única transação (`@Transactional`), garantindo que qualquer falha durante o processo não deixe registro parcial visível (FR-006)
- [ ] T011 [US1] Adicionar validação de unicidade de `identificadorEmpresa` em `TenantService.cadastrar(...)`, lançando exceção de conflito tratada como HTTP 409 (depende de T009; FR-003, Edge Case 1)
- [ ] T012 [US1] Adicionar validação de `planoId` obrigatório e existente em `TenantService.cadastrar(...)`, lançando exceção tratada como HTTP 400 quando ausente/inválido (depende de T009; FR-002)
- [ ] T013 [US1] Implementar `TenantController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/controller/TenantController.java` com `POST /api/v1/plataforma/tenants` (retorna 201) e `GET /api/v1/plataforma/tenants/{id}` (retorna 200 ou 404), restrito ao perfil de administrador da plataforma (depende de T008, T009)
- [ ] T014 [US1] Adicionar resposta 403 no `TenantController` quando o usuário autenticado não for administrador da plataforma (depende de T013)
- [ ] T015 [P] [US1] Criar tela de cadastro de empresa assinante em `mile_bag_app/src/features/back-office/tenant-onboarding/CadastroTenantForm.tsx`, consumindo `POST /api/v1/plataforma/tenants` e exibindo seletor de plano existente
- [ ] T016 [US1] Adicionar tratamento de erro na UI para 400 (plano ausente/inválido) e 409 (identificador duplicado) em `CadastroTenantForm.tsx` (depende de T015, T011, T012)
- [ ] T017 [US1] Criar tela/consulta de confirmação pós-cadastro (Acceptance Scenario 3 — nenhuma pendência de infraestrutura) em `mile_bag_app/src/features/back-office/tenant-onboarding/TenantCadastradoConfirmacao.tsx`, consumindo `GET /api/v1/plataforma/tenants/{id}` (depende de T013, T015)

**Checkpoint**: Cadastro de tenant funcional e isolado de ponta a ponta — testável de forma independente, sem depender de HU15 ou HU16 estarem implementadas (apenas de `PlanoAssinatura` já existir como entidade referenciável, ver Dependencies abaixo).

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T018 [P] Escrever teste automatizado de isolamento entre tenants (tentativa de leitura cross-tenant logo após o cadastro) — critério de sucesso SC-002 — em `mile_bag_api/src/test/java/.../business/tenant/TenantIsolationTest.java`, usando Testcontainers contra PostgreSQL real
- [ ] T019 Revisar mensagens de erro de 409/400 para não vazar dados de outro tenant (ex.: não expor nome da empresa já cadastrada com o mesmo identificador, só o fato do conflito)
- [ ] T020 Executar a validação descrita em [quickstart.md](./quickstart.md)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente.
- **Foundational (Phase 2)**: Depende da Phase 1 — BLOQUEIA toda a Phase 3.
- **User Story 1 (Phase 3)**: Depende da Phase 2 completa.
- **Polish (Final Phase)**: Depende da Phase 3 completa.

### Dependência entre features (fora desta HU)

- Esta HU é **fundacional**: [HU15](../HU15-convidar-primeiro-administrador-tenant/spec.md) (convite ao primeiro administrador) e [HU16](../HU16-gerenciar-planos-assinatura/spec.md) (planos de assinatura) dependem dela — T009 assume que já existe ao menos um `PlanoAssinatura` cadastrável/referenciável (entidade definida em HU16). Se HU16 ainda não estiver implementada, criar `PlanoAssinatura` como entidade mínima (id, nome) apenas para satisfazer a referência de T009, sem as regras completas de HU16.
- HU15 não pode começar sua Phase 3 até que um `Tenant` exista (produzido por esta HU) — dependência de dado, não de código.

### Parallel Opportunities

- T001 e T002 (Setup) podem rodar em paralelo.
- T008 e T015 podem rodar em paralelo, mas T015 só é útil de fato após T013 (endpoint) existir.
- T018 e T019 (Polish) podem rodar em paralelo.

---

## Parallel Example: User Story 1

```bash
# Back-end (DTOs) e front-end (tela) podem começar em paralelo, já que dependem de arquivos diferentes:
Task: "Criar CadastroTenantRequestDTO/TenantResponseDTO em mile_bag_api/.../dto/"
Task: "Criar tela de cadastro em mile_bag_app/src/features/back-office/tenant-onboarding/CadastroTenantForm.tsx"
```

---

## Implementation Strategy

### MVP First (User Story 1 única)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRITICAL — inclui a política de RLS genérica reutilizada por todo o sistema)
3. Completar Phase 3: User Story 1
4. **STOP and VALIDATE**: rodar [quickstart.md](./quickstart.md) e confirmar os 4 cenários de aceite do spec
5. Entregar/demonstrar — esta HU é o pré-requisito de dado para HU15

### Incremental Delivery

Como esta HU tem uma única User Story, a entrega é binária (T001–T020 completas). A próxima fatia de valor no roadmap é HU15 (convite ao primeiro administrador), que consome o tenant criado aqui.

---

## Notes

- [P] = arquivos diferentes, sem dependência entre si.
- [US1] mapeia a tarefa à única User Story desta HU.
- Sem tarefas de teste funcional automatizado além do teste de isolamento (T018, que é um requisito de segurança explícito do spec, não um teste de comportamento geral).
- Fazer commit após cada tarefa ou grupo lógico coeso.
- Parar no checkpoint da Phase 3 para validar a história de forma independente antes de seguir para o Polish.
