---

description: "Task list for HU15 — Convidar o primeiro administrador do tenant"
---

# Tasks: HU15 — Convidar o primeiro administrador do tenant

**Input**: Design documents from `mile_bag_documentacao/specs/HU15-convidar-primeiro-administrador-tenant/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/convite-api.md](./contracts/convite-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — omitidos.

**Organization**: HU15 tem uma única User Story (P1). Todas as tarefas de implementação carregam `[US1]`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 desta HU
- Caminhos de arquivo exatos, conforme `plan.md` → Project Structure

---

## Phase 1: Setup

**Purpose**: Preparar o envio de e-mail transacional e a estrutura de arquivos desta HU dentro do domínio `tenant` já existente (de HU14).

- [ ] T001 Adicionar dependência de cliente de e-mail transacional (SMTP ou equivalente gratuito, conforme decisão a registrar em `research.md` desta HU) no `mile_bag_api/pom.xml`
- [ ] T002 [P] Adicionar propriedade de configuração `milebag.convite.prazo-validade` (ex.: horas) em `mile_bag_api/src/main/resources/application.yml`, com valor default documentado (parametrização — não constante fixa no código, conforme Assumptions do spec)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persistência do convite e geração segura de token — pré-requisito bloqueante de toda a User Story.

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase.

- [ ] T003 Criar migração Flyway para a tabela `convite_administrador` (`token` identificador único opaco, `tenant_id` referência, `email_destinatario`, `status` enum `PENDENTE|USADO|EXPIRADO|INVALIDADO`, `criado_em`, `expira_em`, `usado_em` nullable) em `mile_bag_api/src/main/resources/db/migration/V<próxima>__create_convite_administrador.sql`, com Row Level Security por `tenant_id`
- [ ] T004 Criar entidade JPA `ConviteAdministrador` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/model/ConviteAdministrador.java`, mapeando os campos de T003, com enum `ConviteStatus` (depende de T003)
- [ ] T005 Criar `ConviteAdministradorRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/repository/ConviteAdministradorRepository.java`, com consulta por `token` e por `tenantId` + `status = PENDENTE` (depende de T004)
- [ ] T006 Implementar gerador de token opaco e não previsível (ex.: UUID v4 criptograficamente aleatório ou equivalente) em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/TokenConviteGenerator.java`

**Checkpoint**: Estrutura de convite pronta para persistir, consultar e gerar tokens seguros — a User Story 1 pode começar.

---

## Phase 3: User Story 1 - Convidar automaticamente o primeiro administrador ao concluir o cadastro do tenant (Priority: P1) 🎯 MVP

**Goal**: Ao concluir o cadastro de um tenant (HU14), disparar automaticamente um convite ao primeiro administrador indicado, permitindo que ele defina credenciais e assuma o papel de administrador do tenant, sem ação manual adicional.

**Independent Test**: Cadastrar um tenant com um e-mail de administrador indicado e verificar que o convite é enviado automaticamente, sem ação manual, e que o link do convite permite ativar o acesso.

### Implementation for User Story 1

- [ ] T007 [US1] Implementar `ConviteAdministradorService.criarEEnviar(tenantId, emailDestinatario)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/ConviteAdministradorService.java`: gera token (T006), calcula `expiraEm` a partir de `milebag.convite.prazo-validade` (T002), persiste com `status = PENDENTE` e dispara o e-mail (depende de T005, T006)
- [ ] T008 [US1] Conectar `ConviteAdministradorService.criarEEnviar(...)` ao final de `TenantService.cadastrar(...)` (de HU14, `mile_bag_api/.../business/tenant/service/TenantService.java`), disparado dentro da mesma operação de cadastro, sem exigir chamada separada (depende de T007; integra com HU14)
- [ ] T009 [US1] Implementar validação do formato do e-mail do administrador indicado antes de disparar o convite em `ConviteAdministradorService.criarEEnviar(...)`, sinalizando pendência no cadastro do tenant quando inválido, sem deixar o tenant sem caminho de ativação (depende de T007; Edge Case 1)
- [ ] T010 [US1] Implementar `ConviteAdministradorController.reenviar(tenantId)` com `POST /api/v1/plataforma/tenants/{tenantId}/convites/reenviar` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/controller/ConviteAdministradorController.java`: invalida qualquer convite `PENDENTE` existente para o tenant e cria um novo via T007 (depende de T007; FR-006, Edge Case 2/3)
- [ ] T011 [US1] Adicionar resposta 409 em `reenviar(...)` quando o tenant já possui administrador ativo, e 403 quando o usuário autenticado não é administrador da plataforma (depende de T010)
- [ ] T012 [US1] Implementar `ConviteAdministradorController.consultar(token)` com `GET /api/v1/convites/{token}` (endpoint público, sem autenticação prévia), retornando 200/404/409/410 conforme o `status` do convite (depende de T005)
- [ ] T013 [US1] Implementar `ConviteAdministradorController.ativar(token, credenciais)` com `POST /api/v1/convites/{token}/ativar`: revalida o estado do convite no momento do envio (não confia no `GET` prévio), cria a conta de Usuário com perfil de administrador associado a `tenantId`, e muda `status` para `USADO` com `usadoEm` preenchido (depende de T005, T012; FR-002, FR-003, Acceptance Scenario 2)
- [ ] T014 [US1] Implementar expiração de convite: ao consultar/ativar um convite com `expiraEm` no passado e `status = PENDENTE`, tratar/atualizar como `EXPIRADO` antes de responder 410, em `ConviteAdministradorService` (depende de T012, T013; FR-005, Acceptance Scenario 4)
- [ ] T015 [US1] Adicionar rejeição de reuso: `ativar(...)` retorna 409/404/410 para qualquer convite com `status` diferente de `PENDENTE`, nunca criando uma segunda conta a partir do mesmo token (depende de T013; FR-004, Acceptance Scenario 3)
- [ ] T016 [P] [US1] Criar tela de ativação de convite em `mile_bag_app/src/features/back-office/tenant-onboarding/AtivarConviteForm.tsx`, consumindo `GET /api/v1/convites/{token}` para validar antes de exibir o formulário e `POST /api/v1/convites/{token}/ativar` para submeter as credenciais
- [ ] T017 [US1] Adicionar tratamento de erro na UI para 404/409/410 em `AtivarConviteForm.tsx`, orientando o destinatário a solicitar um novo convite quando expirado (depende de T016, T014)

**Checkpoint**: Convite automático, ativação de conta e reenvio funcionais de ponta a ponta — testável de forma independente, desde que um Tenant já exista (produzido por HU14).

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T018 [P] Adicionar job/rotina de marcação de convites expirados em lote (para consultas administrativas futuras, sem depender apenas da checagem lazy de T014) em `mile_bag_api/src/main/java/com/marillia/milebag/business/tenant/service/ConviteExpiracaoScheduler.java`
- [ ] T019 Revisar mensagens de erro do endpoint público `GET/POST /api/v1/convites/{token}` para não vazar informação sobre a existência de outros tenants/convites além do token consultado
- [ ] T020 Executar a validação descrita em [quickstart.md](./quickstart.md)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente.
- **Foundational (Phase 2)**: Depende da Phase 1 — BLOQUEIA toda a Phase 3.
- **User Story 1 (Phase 3)**: Depende da Phase 2 completa.
- **Polish (Final Phase)**: Depende da Phase 3 completa.

### Dependência entre features (fora desta HU)

Esta HU **depende diretamente de [HU14](../HU14-cadastrar-empresa-assinante/spec.md)**: T008 só pode ser implementada depois que `TenantService.cadastrar(...)` (HU14) existir, pois o convite é disparado como parte dessa mesma operação. Não há tarefa cruzada duplicada aqui — HU14 é pré-requisito de dado e de código para a Phase 3 desta HU. Recomenda-se concluir as tarefas de HU14 (T001–T017 do tasks.md dela) antes de iniciar T008.

### Parallel Opportunities

- T001 e T002 (Setup) podem rodar em paralelo.
- T012 e T013 dependem ambos de T005, mas são endpoints distintos e podem ser implementados em paralelo por desenvolvedores diferentes.
- T016 pode começar em paralelo com T012/T013, mas só é testável de ponta a ponta após eles existirem.
- T018 e T019 (Polish) podem rodar em paralelo.

---

## Parallel Example: User Story 1

```bash
# Consulta e ativação de convite são endpoints distintos, arquivos/métodos diferentes:
Task: "Implementar GET /api/v1/convites/{token} em ConviteAdministradorController.consultar"
Task: "Implementar POST /api/v1/convites/{token}/ativar em ConviteAdministradorController.ativar"
```

---

## Implementation Strategy

### MVP First (User Story 1 única)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRITICAL — inclui geração segura de token)
3. Completar Phase 3: User Story 1 (após HU14 estar disponível — ver Dependencies)
4. **STOP and VALIDATE**: rodar [quickstart.md](./quickstart.md) e confirmar os 4 cenários de aceite do spec
5. Entregar/demonstrar — fecha o fluxo de onboarding iniciado por HU14

### Incremental Delivery

Como esta HU tem uma única User Story, a entrega é binária (T001–T020 completas). Junto com HU14, completa o fluxo de onboarding de um novo tenant descrito na User Story 1 da spec consolidada (`001-milebag-user-stories`).

---

## Notes

- [P] = arquivos diferentes, sem dependência entre si.
- [US1] mapeia a tarefa à única User Story desta HU.
- Sem tarefas de teste automatizado — não solicitadas no spec.
- Fazer commit após cada tarefa ou grupo lógico coeso.
- Parar no checkpoint da Phase 3 para validar a história de forma independente antes de seguir para o Polish.
