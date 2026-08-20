# Implementation Plan: HU15 — Convidar o primeiro administrador do tenant

**Branch**: `HU15-convidar-primeiro-administrador-tenant` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU15-convidar-primeiro-administrador-tenant/spec.md`

## Summary

Reação automática ao término do cadastro de um tenant (HU14): gerar um convite com token de uso único e prazo de validade, enviá-lo ao destinatário indicado, e permitir a ativação da conta de administrador a partir do link do convite. Fecha o fluxo de onboarding iniciado por HU14.

## Technical Context

Stack completa em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context). Específico desta HU:

**Primary Dependencies**: Spring Security (para o fluxo de criação de credenciais na ativação do convite); serviço de envio de e-mail transacional — a evidência do TAP não especifica um provedor; tratado como decisão de infraestrutura a resolver em `research.md` desta HU, respeitando a restrição de custo zero do piloto (RNF08 do spec-mãe).

**Constraints**: o token de convite deve ser de uso único e expirável (FR-004, FR-005) — não pode ser um identificador previsível (ex.: sequencial).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates avaliados em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#constitution-check). Reavaliação específica desta HU:

| Gate | Avaliação |
|---|---|
| Segurança e privacidade por padrão | PASS — token de uso único, expirável, e invalidação de convites antigos ao reenviar (FR-004, FR-005, FR-006) evitam acesso indevido por convite vazado ou reaproveitado |
| Confiabilidade e rastreabilidade dos dados | PASS — o estado do convite (pendente/usado/expirado/invalidado) é rastreável, permitindo auditar como cada conta de administrador foi criada |
| Simplicidade antes de generalidade | PASS — um único mecanismo de convite serve ao primeiro administrador; convites para administradores adicionais ficam fora do escopo (ver Assumptions do spec), evitando generalizar prematuramente |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU15-convidar-primeiro-administrador-tenant/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
```

### Source Code (trechos relevantes a esta HU)

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/
└── tenant/
    ├── controller/       # endpoint de ativação do convite (público, autenticado só pelo token)
    ├── service/          # disparo automático do convite ao final do cadastro (HU14 → HU15)
    ├── repository/
    └── model/            # entidade ConviteAdministrador

mile_bag_app/src/features/back-office/
└── tenant-onboarding/     # tela de ativação de convite (compartilhada com HU14)
```

**Structure Decision**: Continua no mesmo subpacote `business/tenant` do plano-mãe e de HU14 — convite é parte do ciclo de onboarding do tenant, não um domínio à parte.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
