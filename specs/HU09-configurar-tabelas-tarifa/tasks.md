---

description: "Task list for HU09 — Configurar tabelas de tarifa"
---

# Tasks: HU09 — Configurar tabelas de tarifa

**Input**: Design documents from `mile_bag_documentacao/specs/HU09-configurar-tabelas-tarifa/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/tarifas-api.md](./contracts/tarifas-api.md), [quickstart.md](./quickstart.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organization**: HU09 tem uma única User Story (P1); todas as tarefas de implementação carregam o rótulo `[US1]`.

## Phase 1: Setup

**Purpose**: Nenhuma dependência nova é necessária — CRUD simples sobre a stack já definida no plano-mãe (Spring Data JPA + PostgreSQL).

- [ ] T001 Confirmar que o subpacote `business/tarifa` existe em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/` (criar `controller/`, `service/`, `repository/`, `model/` se ainda não existirem)

## Phase 2: Foundational

**Purpose**: Estrutura de dados e política de isolamento que bloqueiam a implementação da história

- [ ] T002 Criar migração Flyway para a tabela `tabela_tarifa` em `mile_bag_api/src/main/resources/db/migration/` com as colunas de [data-model.md](./data-model.md) (`id`, `tenant_id`, `escopo`, `motorista_id`, `regiao`, `valor_fixo`, `valor_por_km`, `vigente_desde`)
- [ ] T003 Adicionar política de Row Level Security para `tabela_tarifa` filtrando por `tenant_id`, na mesma migração ou em migração subsequente em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T004 Criar constraint de unicidade condicional (um `PADRAO` vigente por tenant; um `MOTORISTA` vigente por motorista; um `REGIAO` vigente por região) na migração de T002, alinhada às regras de validação de [data-model.md](./data-model.md)

**Checkpoint**: Schema e isolamento prontos — a implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Configurar tabelas de tarifa do tenant (Priority: P1) 🎯 MVP

**Goal**: Permitir que o administrador do tenant cadastre tarifa padrão, por motorista e por região, com resolução automática de especificidade (motorista > região > padrão) no momento do fechamento.

**Independent Test**: Cadastrar uma tarifa padrão do tenant e, em seguida, tarifas específicas por motorista e por região, verificando que cada uma se sobrepõe corretamente à padrão quando aplicável — sem depender do fechamento (HU12) estar implementado.

### Implementation for User Story 1

- [ ] T005 [P] [US1] Criar entidade `TabelaTarifa` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/model/TabelaTarifa.java` com os campos de [data-model.md](./data-model.md) (`id`, `tenantId`, `escopo` como enum `EscopoTarifa` [`PADRAO`, `MOTORISTA`, `REGIAO`], `motoristaId`, `regiao`, `valorFixo`, `valorPorKm`, `vigenteDesde`)
- [ ] T006 [P] [US1] Criar `TabelaTarifaRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/repository/TabelaTarifaRepository.java` (Spring Data JPA) com métodos de busca por `tenantId`, por `tenantId + escopo`, por `tenantId + motoristaId`, por `tenantId + regiao`
- [ ] T007 [US1] Implementar `TabelaTarifaService` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/service/TabelaTarifaService.java` com o método de criação que valida FR-005 (rejeitar se `valorFixo` e `valorPorKm` estiverem ambos nulos) e FR-002/FR-003 (exigir `motoristaId` quando `escopo=MOTORISTA` e `regiao` quando `escopo=REGIAO`) — depende de T005, T006
- [ ] T008 [US1] Implementar no `TabelaTarifaService` o método `resolverTarifaAplicavel(tenantId, motoristaId, regiao)` que aplica a ordem de especificidade motorista > região > padrão (FR-004) — depende de T007
- [ ] T009 [US1] Implementar métodos de listagem, detalhe, atualização e remoção no `TabelaTarifaService`, garantindo que toda consulta é escopada ao tenant autenticado (FR-006) — depende de T007
- [ ] T010 [US1] Criar `TabelaTarifaController` em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/controller/TabelaTarifaController.java` implementando os 5 endpoints de [contracts/tarifas-api.md](./contracts/tarifas-api.md) (`POST /api/v1/tarifas`, `GET /api/v1/tarifas`, `GET /api/v1/tarifas/{id}`, `PUT /api/v1/tarifas/{id}`, `DELETE /api/v1/tarifas/{id}`) — depende de T008, T009
- [ ] T011 [US1] Mapear as respostas de erro do controller conforme [contracts/tarifas-api.md](./contracts/tarifas-api.md): `400` (validação de FR-005 e de campos condicionais), `404` (tarifa inexistente ou de outro tenant, sem vazar existência), `409` (tarifa vigente conflitante na criação; tarifa referenciada por fechamento emitido na remoção) — depende de T010
- [ ] T012 [P] [US1] Criar tela de listagem/cadastro de tarifas em `mile_bag_app/src/features/back-office/tarifas/` (componentes, hook `useTarifas`, `services/tarifasApi.ts` consumindo os endpoints de [contracts/tarifas-api.md](./contracts/tarifas-api.md))
- [ ] T013 [US1] Implementar no front-end a distinção visual entre tarifa padrão, por motorista e por região, e a mensagem de erro quando nem `valorFixo` nem `valorPorKm` são informados (FR-005) — depende de T012

**Checkpoint**: User Story 1 (HU09) completa e testável de forma independente — administrador do tenant consegue cadastrar e consultar tarifas padrão/motorista/região com isolamento entre tenants garantido

---

## Final Phase: Polish & Cross-Cutting Concerns

- [ ] T014 [P] Adicionar logging das operações de criação/atualização/remoção de tarifa em `mile_bag_api/src/main/java/com/marillia/milebag/business/tarifa/service/TabelaTarifaService.java`
- [ ] T015 Executar a validação de [quickstart.md](./quickstart.md) (cadastro de tarifa padrão, por motorista, por região, listagem, isolamento entre tenants e rejeição de tarifa sem valores)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente.
- **Foundational (Phase 2)**: Depende do Setup — BLOQUEIA a User Story 1.
- **User Story 1 (Phase 3)**: Depende da conclusão da Fase 2.
- **Polish (Final Phase)**: Depende da conclusão da Fase 3.

### Dependência entre features

HU12 (Gerar o fechamento quinzenal) consome `TabelaTarifaService.resolverTarifaAplicavel` (T008) no cálculo do pagamento — HU09 deve estar implementada antes da implementação de HU12 (ver [tasks.md de HU12](../HU12-gerar-fechamento-quinzenal/tasks.md)). Esta HU (HU09), isoladamente, não depende de nenhuma outra.

### Within User Story 1

- Modelo (T005) e repositório (T006) antes do serviço (T007).
- Serviço (T007–T009) antes do controller (T010–T011).
- Back-end (T005–T011) antes da tela de front-end (T012–T013), pois o front-end consome os endpoints.

### Parallel Opportunities

- T005 e T006 podem rodar em paralelo (arquivos diferentes, sem dependência entre si).
- T012 pode começar em paralelo ao back-end assim que o contrato de API estiver estável (não depende de T005–T011 estarem *implementados*, só do contrato já definido em `contracts/tarifas-api.md`), mas T013 depende de T012.

---

## Parallel Example: User Story 1

```bash
# Modelo e repositório em paralelo:
Task: "Criar entidade TabelaTarifa em mile_bag_api/.../model/TabelaTarifa.java"
Task: "Criar TabelaTarifaRepository em mile_bag_api/.../repository/TabelaTarifaRepository.java"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup.
2. Completar Phase 2: Foundational (schema + RLS) — bloqueante.
3. Completar Phase 3: User Story 1.
4. **Parar e validar**: rodar [quickstart.md](./quickstart.md) de ponta a ponta.
5. Prosseguir para HU12, que depende desta HU estar concluída.

---

## Notes

- `[P]` = arquivos diferentes, sem dependência entre as tarefas marcadas.
- `[US1]` mapeia a tarefa à única User Story desta HU.
- Commit após cada tarefa ou grupo lógico coeso.
- Evitar: tarefas vagas, conflito no mesmo arquivo, dependências que quebrem a testabilidade independente da história.
