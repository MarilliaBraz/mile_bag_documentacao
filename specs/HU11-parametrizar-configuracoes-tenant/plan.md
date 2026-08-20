# Implementation Plan: HU11 — Parametrizar configurações do tenant

**Branch**: `HU11-parametrizar-configuracoes-tenant` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/HU11-parametrizar-configuracoes-tenant/spec.md`

## Summary

Cinco pequenos cadastros de configuração por tenant (motivos de ocorrência, campos obrigatórios do formulário de baixa, bases aeroportuárias, companhias aéreas atendidas, periodicidade de fechamento), todos consumidos por outras HUs (ocorrência, comprovante/baixa, fechamento). Faz parte do domínio `configuracao` já mapeado no [plano-mãe](../001-milebag-user-stories/plan.md).

## Technical Context

Stack completa já definida no [Technical Context do plano-mãe](../001-milebag-user-stories/plan.md#technical-context) — não repetida aqui. Específico desta HU: nenhuma dependência nova; são tabelas de configuração simples, lidas com frequência (a cada abertura de formulário no app do motorista) — candidatas a cache de leitura por tenant no back-end, se o desempenho percebido exigir (não é um requisito, apenas uma nota de implementação).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Mesmos gates avaliados no [plano-mãe](../001-milebag-user-stories/plan.md#constitution-check), reavaliados para esta HU:

| Gate | Avaliação para HU11 |
|---|---|
| Os 3 pilares: parametrização | PASS — é a própria funcionalidade que implementa a parametrização de comportamento por tenant (motivos, campos, periodicidade), nunca hardcoded |
| Separação `core`/`business` (backend) | PASS — subpacote `business/configuracao` autocontido; nenhuma regra de outro domínio (ocorrência, fechamento) é duplicada aqui, apenas referenciada |
| Isolamento entre tenants (RNF01) | PASS — todas as cinco configurações carregam coluna de tenant, sujeitas à mesma RLS do restante do domínio |

Nenhuma violação identificada.

## Project Structure

Caminhos relevantes dentro dos repositórios já descritos no [plano-mãe](../001-milebag-user-stories/plan.md#project-structure):

```text
mile_bag_api/src/main/java/com/marillia/milebag/business/configuracao/
├── controller/     # endpoints das 5 categorias de configuração (ver contracts/)
├── service/        # regras de desativação (não exclusão) de motivo de ocorrência; troca de periodicidade só no próximo período
├── repository/
└── model/          # MotivoOcorrencia, CampoBaixaConfig, BaseAeroportuaria, CompanhiaAerea, PeriodicidadeFechamento (ver data-model.md)

mile_bag_app/src/features/back-office/configuracao-tenant/    # telas de configuração (HU11)
```

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada.*
