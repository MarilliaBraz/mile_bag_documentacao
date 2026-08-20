# Fase 0 — Pesquisa e Decisões Técnicas

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Isolamento multi-tenant e operação offline seguem as decisões já registradas em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md) (seções 3 e 6). Esta HU acrescenta uma única decisão nova: como representar o catálogo de motivos de ocorrência.

## 1. Catálogo de motivos de ocorrência por tenant

**Decision**: O catálogo de motivos de ocorrência é modelado como dado de referência (lookup) escopado por tenant — uma tabela própria, editável pelo administrador do tenant (HU11), e não um enum fixo no código-fonte do backend.

**Rationale**: RD07 (`../../requisitos/04-regras-dominio.md`) define a ocorrência como "evento padronizado (de um catálogo configurável por tenant)". Um enum fixo violaria diretamente esse requisito e o pilar de parametrização de `CONVENTIONS.md` §2.3 — cada tenant precisa poder ter seus próprios motivos, sem alteração de código ou reimplantação.

**Alternatives considered**: Um catálogo global compartilhado por todos os tenants, com apenas ativação/desativação por tenant, foi considerado, mas descartado por ser menos flexível do que o texto da fonte sugere (motivos "configuráveis" por tenant, não apenas filtráveis) — decisão final de granularidade fica para a implementação de HU11, sem impacto no contrato desta HU (que só consome o catálogo já cadastrado).
