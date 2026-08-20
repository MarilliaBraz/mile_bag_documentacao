# Implementation Plan: Operar offline e sincronizar automaticamente

**Branch**: `HU08-operar-offline-sincronizar` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU08-operar-offline-sincronizar/spec.md`

## Summary

Capacidade transversal que permite à PWA do motorista funcionar sem conexão, enfileirando localmente toda ação de captura (HU01–HU07) e sincronizando-a de forma automática, ordenada e idempotente quando a conectividade retorna. Detalha, especificamente para esta HU, o mecanismo de fila local e o contrato de sincronização em lote — a stack de PWA/offline já está decidida no plano-mãe.

## Technical Context

Herda o Technical Context do plano-mãe ([`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context)): PWA em React/TypeScript com Service Worker via Workbox e fila local em IndexedDB via Dexie.js.

**Constraints específicas desta HU**:
- A fila local deve suportar, no mínimo, um dia inteiro de entregas de um motorista sem perda de dados (ver `../001-milebag-user-stories/research.md`, seção 2).
- A sincronização deve ser idempotente (reenvio seguro) e preservar ordem cronológica (FR-005, FR-006).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Avaliação para esta HU |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — cada evento sincronizado carrega identificador próprio e timestamp de criação local, preservando a ordem real dos fatos, não a ordem de chegada ao servidor |
| Segurança e privacidade por padrão | PASS — a fila local no dispositivo do motorista só contém dados das próprias entregas; a sincronização passa pela mesma autenticação/isolamento de tenant do restante da API |
| Separação `core`/`business` (backend) | PASS — o endpoint de sincronização em lote entra em `core` (é infraestrutura genérica de ingestão, não regra de negócio de um domínio específico), delegando a cada evento para o `business/<dominio>` correspondente |
| Organização por `feature` (frontend) | PASS — a fila local e a lógica de sincronização entram como `features/motorista/offline-sync`, uma feature transversal consumida pelas demais features do motorista |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU08-operar-offline-sincronizar/
├── spec.md
├── plan.md          # este arquivo
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes)

```text
mile_bag_api/src/main/java/com/marillia/milebag/core/sync/
└── (endpoint de sincronização em lote, deduplicação por idempotency key, roteamento de cada evento para o business/<domínio> correspondente)

mile_bag_app/src/features/motorista/offline-sync/
├── fila-local/          # Dexie.js — schema da fila de eventos pendentes
├── service-worker/      # Workbox — detecção de conectividade, disparo de sincronização
└── indicador-status/    # UI de "offline" / "pendente de sincronização" (RI05)
```

**Structure Decision**: O endpoint de sincronização em lote entra em `core/sync` (não em um `business/<domínio>` específico) porque é um mecanismo de ingestão genérico, reaproveitado por todos os domínios (bdo, entrega, rota, comprovante, ocorrência) — cada evento do lote é roteado internamente para o serviço de domínio correspondente, sem duplicar lógica de negócio em `core`. No front-end, a fila e a sincronização formam uma feature própria e transversal, consumida pelas demais features do motorista.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
