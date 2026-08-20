---

description: "Task list for HU08 — Operar offline e sincronizar automaticamente"
---

# Tasks: Operar offline e sincronizar automaticamente

**Input**: Design documents from `mile_bag_documentacao/specs/HU08-operar-offline-sincronizar/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [data-model.md](./data-model.md), [contracts/sincronizacao-api.md](./contracts/sincronizacao-api.md), [quickstart.md](./quickstart.md), [research.md](./research.md)

**Tests**: Não solicitados no spec — nenhuma tarefa de teste incluída.

**Organização**: Esta HU tem uma única User Story (US1, P1), mas é transversal — a fila local e o endpoint de sincronização aqui construídos são consumidos por HU01–HU07. Fases: Setup → Foundational → US1 → Polish.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependência entre si)
- **[US1]**: Tarefa pertence à User Story 1 (única história desta HU)

## Path Conventions

- Back-end: `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/`
- Front-end (PWA motorista, infraestrutura offline compartilhada): `mile_bag_app/src/features/motorista/offline-sync/`

---

## Phase 1: Setup

- [ ] T001 Confirmar/criar o pacote `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/` no back-end
- [ ] T002 Adicionar Dexie.js ao `mile_bag_app` (se ainda não presente) e criar o módulo de banco local em `mile_bag_app/src/features/motorista/offline-sync/db.ts`, declarando a tabela `registrosPendentes` com os campos de `data-model.md` (`idLocal`, `sequenciaLocal`, `tipoEvento`, `payload`, `timestampCriacaoLocal`, `status`, `tentativas`)
- [ ] T003 [P] Configurar o Service Worker via Workbox em `mile_bag_app` para detecção de eventos `online`/`offline` do navegador, expondo um hook/store de conectividade em `mile_bag_app/src/features/motorista/offline-sync/useConectividade.ts`

---

## Phase 2: Foundational (Blocking Prerequisites)

**⚠️ CRITICAL**: Nenhuma tarefa da Phase 3 pode começar antes desta fase estar completa

- [ ] T004 Criar migração Flyway para a tabela `evento_sincronizado` (`id_local` [único por tenant/motorista], `tipo_evento`, `processado_em`, `resultado`, coluna de tenant, política de Row Level Security) em `mile_bag_api/src/main/resources/db/migration/`
- [ ] T005 Implementar gerador de `sequenciaLocal` monotônico por dispositivo (contador local persistido, não o relógio do aparelho — conforme `research.md`) em `mile_bag_app/src/features/motorista/offline-sync/sequenciaLocal.ts`, usado por todo evento gravado na fila (depende de T002)
- [ ] T006 Implementar a função `enfileirar(tipoEvento, payload)` — grava um `RegistroPendente` com `idLocal` (UUID), `sequenciaLocal` (T005) e `status = PENDENTE` na tabela Dexie — em `mile_bag_app/src/features/motorista/offline-sync/fila.ts` (depende de T002, T005). Esta função é o ponto de integração que HU01–HU07 devem chamar quando a ação for executada sem conexão

**Checkpoint**: Fila local, detecção de conectividade e schema de deduplicação no servidor prontos — implementação da User Story 1 pode começar

---

## Phase 3: User Story 1 - Continuar trabalhando sem conexão e sincronizar automaticamente (Priority: P1) 🎯 MVP

**Goal**: Permitir que o motorista execute todas as ações de captura (HU01–HU07) sem conexão, com sincronização automática, ordenada e idempotente assim que a conectividade retornar, sem intervenção manual.

**Independent Test**: Colocar o dispositivo em modo avião, executar o ciclo completo de uma entrega (captura, atribuição, rota, viagem, comprovante) gerando eventos na fila local, restaurar a conectividade e verificar que todos os registros chegam ao servidor via `POST /api/v1/sync/eventos`, sem intervenção manual e sem duplicação.

### Implementation for User Story 1 — Back-end (processamento do lote)

- [ ] T007 [US1] Implementar `EventoSincronizadoRepository` em `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/EventoSincronizadoRepository.java`, com busca por `idLocal` para deduplicação (FR-006)
- [ ] T008 [US1] Implementar `SincronizacaoService.processarLote(eventos)` em `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/SincronizacaoService.java`: itera os eventos na ordem recebida (já ordenados por `sequenciaLocal` pelo cliente, FR-005), para cada um verifica se `idLocal` já existe em `EventoSincronizado` (retorna `DUPLICADO` sem reprocessar, FR-006), caso contrário despacha para o handler do domínio correspondente ao `tipoEvento` (T009) em processamento *melhor esforço item a item* — uma falha em um item não interrompe os demais (depende de T007)
- [ ] T009 [US1] Implementar `SincronizacaoEventoDispatcher` em `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/SincronizacaoEventoDispatcher.java`, mapeando cada `tipoEvento` para o serviço de domínio equivalente já implementado em HU01–HU07: `BDO_CAPTURADO` → `BdoService`, `ENTREGA_AUTOATRIBUIDA` → `EntregaService`, `ROTA_ESCOLHIDA` → `RotaService`, `VIAGEM_INICIADA` → `ViagemService` (HU04), `BAGAGEM_CARONA_VINCULADA` → `BagagemCaronaService`, `COMPROVANTE_REGISTRADO` → `ComprovanteEntregaService`, `OCORRENCIA_REGISTRADA` → `OcorrenciaService` — reaproveitando a mesma lógica de validação síncrona (depende de T008; integra com HU01–HU07)
- [ ] T010 [US1] Implementar endpoint `POST /api/v1/sync/eventos` em `mile_bag_api/src/main/java/com/marillia/milebag/business/sincronizacao/SincronizacaoController.java`, conforme `contracts/sincronizacao-api.md`: sempre `200 OK` com resultado por item, exceto erros de nível de requisição (`400`/`401`) (depende de T008, T009)
- [ ] T011 [US1] Implementar endpoint `GET /api/v1/sync/status` no mesmo controller, conforme `contracts/sincronizacao-api.md` (depende de T007)

### Implementation for User Story 1 — Front-end (fila e disparo automático)

- [ ] T012 [US1] Implementar `sincronizar()` em `mile_bag_app/src/features/motorista/offline-sync/sincronizador.ts`: lê da fila local todos os `RegistroPendente` com `status = PENDENTE` ou `ERRO`, ordenados por `sequenciaLocal`, monta o lote e chama `POST /api/v1/sync/eventos` (depende de T002, T006, T010)
- [ ] T013 [US1] Processar a resposta de `sincronizar()`: marcar como `SINCRONIZADO` (e remover ou arquivar) os itens com resultado `ACEITO`/`DUPLICADO`; incrementar `tentativas` e manter `status = ERRO` os itens com `resultado: "ERRO"`, para nova tentativa (FR-007) em `mile_bag_app/src/features/motorista/offline-sync/sincronizador.ts` (depende de T012)
- [ ] T014 [US1] Disparar `sincronizar()` automaticamente ao detectar transição para online (via `useConectividade`, T003) e, como reforço, em intervalo periódico enquanto online (retry de itens `ERRO`), sem exigir ação do motorista (FR-004) em `mile_bag_app/src/features/motorista/offline-sync/useSincronizacaoAutomatica.ts` (depende de T003, T012, T013)
- [ ] T015 [US1] Implementar indicador de UI: banner/ícone de "operando offline" quando `useConectividade` reportar offline, e contagem/lista de registros pendentes de sincronização (FR-002, FR-003) em `mile_bag_app/src/features/motorista/offline-sync/IndicadorSincronizacao.tsx` (depende de T003, T006)
- [ ] T016 [US1] Integrar a função `enfileirar()` (T006) como fallback nos clients de API já criados por HU01–HU07 (captura de BDO, autoatribuição, rota, viagem, bagagem carona, comprovante, ocorrência): quando `useConectividade` indicar offline, chamar `enfileirar(tipoEvento, payload)` em vez do endpoint síncrono correspondente (depende de T003, T006; integra com HU01–HU07)
- [ ] T017 [US1] Garantir, na UI de histórico/detalhe de uma entrega, que eventos ainda pendentes de sincronização apareçam marcados como tal (não confundidos com eventos já confirmados pelo servidor), reforçando FR-003 em `mile_bag_app/src/features/motorista/offline-sync/IndicadorSincronizacao.tsx` (depende de T015)

**Checkpoint**: Neste ponto, a User Story 1 desta HU deve estar completa e testável de forma independente via `quickstart.md`

---

## Phase 4: Polish & Cross-Cutting Concerns

- [ ] T018 [P] Adicionar teste automatizado de isolamento entre tenants/motoristas para `EventoSincronizado` (RNF01) — um `idLocal` de um motorista não deve poder ser confundido com o de outro
- [ ] T019 [P] Adicionar tratamento para acúmulo prolongado de registros pendentes (edge case do spec.md): garantir que a fila local não tenha limite artificial de tamanho e que o armazenamento local não descarte itens silenciosamente
- [ ] T020 Run quickstart.md validation — executar o roteiro de [`quickstart.md`](./quickstart.md) de ponta a ponta, incluindo o cenário de reconexão interrompida no meio do envio de um lote (edge case do spec.md)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Sem dependências — pode começar imediatamente
- **Foundational (Phase 2)**: Depende da conclusão do Setup — BLOQUEIA a User Story 1
- **User Story 1 (Phase 3)**: Depende da conclusão da Phase 2
- **Polish (Phase 4)**: Depende da conclusão da Phase 3

### Within User Story 1

- Back-end (T007–T011) e a base da fila local (T002, T005, T006, já na Phase 2) são pré-requisitos de T012 (que efetivamente chama o endpoint)
- T009 (dispatcher) depende dos serviços de domínio de HU01–HU07 já existirem — se ainda não existirem no ambiente, implementar como stubs e substituir depois
- T012 → T013 → T014 (cadeia: enviar lote → processar resposta → disparo automático)
- T016 (integração com os fluxos de HU01–HU07) depende de T003 e T006, mas não bloqueia o restante desta HU — pode ser feito em paralelo por quem implementa cada HU consumidora

### Parallel Opportunities

- T003 pode rodar em paralelo a T002 (Service Worker vs. schema Dexie)
- T007 pode rodar em paralelo a T002/T003 (back-end vs. front-end)
- T015 pode rodar em paralelo a T012/T013 (UI de indicador vs. lógica de sincronização)
- T018 e T019 podem rodar em paralelo entre si e com T020

---

## Parallel Example: User Story 1

```bash
# Lançar em paralelo:
Task: "Implementar EventoSincronizadoRepository em mile_bag_api/.../sincronizacao/EventoSincronizadoRepository.java"
Task: "Criar módulo db.ts (Dexie) em mile_bag_app/.../offline-sync/db.ts"
Task: "Implementar IndicadorSincronizacao.tsx em mile_bag_app/.../offline-sync/"
```

---

## Implementation Strategy

### MVP First (única User Story)

1. Completar Phase 1: Setup
2. Completar Phase 2: Foundational (CRÍTICO — fila local e deduplicação no servidor)
3. Completar Phase 3: User Story 1
4. **PARAR e VALIDAR**: testar a User Story 1 de forma independente via `quickstart.md`, incluindo o cenário offline completo
5. Completar Phase 4: Polish

### Notas

- Esta HU é transversal: T009 e T016 assumem que os serviços/domínios de HU01–HU07 já existem (ou existirão em paralelo) — se implementada antes delas, criar stubs mínimos para T009 e substituir quando cada HU estiver pronta.
- Perda de dados por troca/reset do dispositivo antes da sincronização é um risco aceito (ver Assumptions do spec.md) — não gerar tarefa de backup fora do dispositivo.
- Commitar após cada tarefa ou grupo lógico de tarefas.
- Evitar: tarefas vagas, conflito de mesmo arquivo entre tarefas paralelas, dependências cruzadas que quebrem a testabilidade independente da história.
