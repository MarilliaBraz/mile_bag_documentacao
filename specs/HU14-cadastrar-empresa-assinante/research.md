# Fase 0 — Pesquisa e Decisões Técnicas: HU14

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

O mecanismo de isolamento multi-tenant (tabelas compartilhadas + Row Level Security + claim de tenant no JWT) já está decidido e justificado em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção **3. Isolamento multi-tenant**. Esta HU é a primeira a colocar esse mecanismo em prática (criação do primeiro registro de um tenant), então o único ponto específico a resolver aqui é a atomicidade do cadastro.

## Cadastro atômico do tenant

**Decision**: A criação do tenant e sua associação ao plano de assinatura ocorrem em uma única transação de banco de dados; qualquer falha no meio do processo desfaz o cadastro por completo (nenhum tenant parcial fica visível).

**Rationale**: Requisito explícito do spec (FR-006, Edge Case "cadastro interrompido") — evitar um estado inconsistente em que o tenant existe mas não tem plano válido, o que quebraria os gates de limite de motoristas/BDOs de HU16 (FR-017 do spec-mãe).

**Alternatives considered**: Criar o tenant primeiro e associar o plano em uma segunda chamada foi descartado — abriria uma janela em que o tenant existe sem plano, violando FR-002.
