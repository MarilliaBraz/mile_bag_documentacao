# Implementation Plan: HU12 — Gerar o fechamento quinzenal

**Branch**: `HU12-gerar-fechamento-quinzenal` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU12-gerar-fechamento-quinzenal/spec.md`

## Summary

Job de agregação que, para um tenant e período, percorre as entregas concluídas, resolve a tarifa aplicável (domínio `tarifa`, [HU09](../HU09-configurar-tabelas-tarifa/plan.md)) e o valor de pedágio congelado no snapshot de cada uma, e emite dois relatórios separados (pagamento de entregas; reembolso de pedágios). Faz parte do domínio `fechamento` já mapeado no [plano-mãe](../001-milebag-user-stories/plan.md).

## Technical Context

Stack completa já definida no [Technical Context do plano-mãe](../001-milebag-user-stories/plan.md#technical-context) — não repetida aqui. Específico desta HU: geração de PDF via Apache PDFBox ou JasperReports Library (decisão registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção 8); a identidade visual por tenant usada nos relatórios (HU13, fora desta pasta) é uma dependência de saída, não de entrada, desta HU.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Mesmos gates avaliados no [plano-mãe](../001-milebag-user-stories/plan.md#constitution-check), reavaliados para esta HU:

| Gate | Avaliação para HU12 |
|---|---|
| Confiabilidade e rastreabilidade dos dados | PASS — cálculo determinístico a partir de dados já congelados (snapshot de rota, tarifa e pedágio resolvidos), sem consulta a serviços externos no momento do fechamento (FR-001, FR-002) |
| Separação `core`/`business` (backend) | PASS — subpacote `business/fechamento` orquestra, mas não duplica, a lógica de `tarifa` e `pedagio` (reuso, não cópia) |
| Isolamento entre tenants (RNF01) | PASS — fechamento e seus itens carregam coluna de tenant; a consulta de entregas do período já é filtrada por tenant via RLS (FR-008) |
| Simplicidade antes de generalidade | PASS — sem agendamento automático nem reprocessamento sofisticado nesta HU (ver Assumptions do spec.md) |

Nenhuma violação identificada.

## Project Structure

Caminhos relevantes dentro dos repositórios já descritos no [plano-mãe](../001-milebag-user-stories/plan.md#project-structure):

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/fechamento/
├── controller/     # endpoint de geração e consulta de fechamento (ver contracts/)
├── service/        # orquestra tarifa.service e pedagio.service; agrega por motorista; gera PDF
├── repository/
└── model/          # Fechamento, ItemFechamento (ver data-model.md)

mile_bag_app/src/features/back-office/fechamento/    # tela de geração e download do fechamento (HU12)
```

**Dependências de implementação**: esta HU só pode ser implementada depois de `business/tarifa` ([HU09](../HU09-configurar-tabelas-tarifa/plan.md)) e `business/pedagio` ([HU10](../HU10-cadastrar-pracas-pedagio/plan.md)), e depende do domínio `entrega`/`rota` (snapshot de viagem) descrito no [plano-mãe](../001-milebag-user-stories/plan.md).

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
