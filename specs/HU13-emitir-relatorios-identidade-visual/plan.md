# Implementation Plan: HU13 — Emitir relatórios com identidade visual do tenant

**Branch**: `HU13-emitir-relatorios-identidade-visual` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU13-emitir-relatorios-identidade-visual/spec.md`

## Summary

Permitir que cada tenant configure logotipo e cabeçalho próprios e que essa identidade seja aplicada automaticamente na geração dos relatórios PDF do fechamento (HU12). É um acréscimo de apresentação sobre uma capacidade já existente (geração de relatório), não uma nova entidade de negócio.

## Technical Context

A stack completa (linguagens, frameworks, banco, infraestrutura) já está detalhada em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#technical-context) — não repetida aqui. Específico desta HU:

**Primary Dependencies**: Apache PDFBox ou JasperReports Library (decisão registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md#8-geração-de-relatórios-do-fechamento) §8) — JasperReports é preferível aqui por facilitar templates visuais parametrizáveis por tenant.

**Storage**: arquivo de logotipo armazenado no mesmo volume de disco com prefixo por tenant usado para imagens de BDO/POD (ver plano-mãe, seção Storage), atrás da mesma interface de repositório.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates avaliados em [`../001-milebag-user-stories/plan.md`](../001-milebag-user-stories/plan.md#constitution-check) (fonte: `mile_bag_audite/CONSTITUTION.md` e `CONVENTIONS.md`). Reavaliação específica desta HU:

| Gate | Avaliação |
|---|---|
| Simplicidade antes de generalidade | PASS — reutiliza a entidade de fechamento existente (HU12); a identidade visual é um atributo de configuração do tenant, não uma nova entidade de domínio |
| Segurança e privacidade por padrão / isolamento entre tenants | PASS — a identidade visual de um tenant nunca deve ser lida ou aplicada ao gerar relatório de outro tenant (FR-003); segue o mesmo mecanismo de RLS + prefixo de armazenamento por tenant do restante do sistema |
| Parametrização (3 pilares) | PASS — identidade visual é configuração por tenant, não hardcoded |

Nenhuma violação identificada.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/HU13-emitir-relatorios-identidade-visual/
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
└── configuracao/                       # pacote já existente (HU11); reaproveitado, sem subpacote novo
    ├── controller/                     # + endpoint de identidade visual (ver contracts/)
    ├── service/                        # + regra de leitura em tempo de renderização (não retroativa)
    ├── repository/
    └── model/                          # + IdentidadeVisualTenant (ver data-model.md)
mile_bag_api/.../business/fechamento/   # consumidor: aplica a identidade ao gerar o PDF (HU12)

mile_bag_app/src/features/back-office/
├── configuracao-tenant/                # tela de upload/edição da identidade visual (compartilhada com HU11)
└── fechamento/                         # exibição/download do relatório já com a identidade aplicada (HU12)
```

**Structure Decision**: Sem pacote novo dedicado — a identidade visual entra em `business/configuracao` (já previsto para parametrização por tenant no plano-mãe) e é consumida por `business/fechamento` no momento da geração do PDF. No front-end, reaproveita a feature `configuracao-tenant` do back-office.

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
