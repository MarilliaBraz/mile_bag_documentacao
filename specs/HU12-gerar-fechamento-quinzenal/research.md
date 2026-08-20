# Fase 0 — Pesquisa e Decisões Técnicas: HU12

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A escolha de biblioteca de geração de PDF (PDFBox / JasperReports) já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção **"8. Geração de relatórios do fechamento"**, e o princípio de nunca recalcular sobre dados congelados está na seção **"7. Congelamento (snapshot) da rota"** do mesmo documento. Esta HU aplica ambos, sem introduzir tecnologia nova.

## Idempotência da geração de fechamento

**Decision**: A geração de um fechamento é idempotente por `(tenantId, período)` — uma chave única no banco impede duas linhas de `Fechamento` para a mesma combinação; uma segunda tentativa retorna o fechamento já existente em vez de recalcular.

**Rationale**: Requisito explícito do spec.md desta HU (FR-005, Acceptance Scenario 5): o fechamento não pode ser gerado em duplicidade. Uma constraint de unicidade no banco é mais robusta do que apenas uma checagem na camada de serviço, que poderia ser burlada por chamadas concorrentes.

**Alternatives considered**: Permitir múltiplos fechamentos por período e escolher "o mais recente" como válido foi descartado — contraria RN08 (fechamento como apuração única e confiável do período) e abriria margem para divergência de valores entre execuções, exatamente o problema que o projeto existe para eliminar (documento de evidência, seção 1.2).

## Entregas sem tarifa ou pedágio configurado

**Decision**: O cálculo por entrega roda de forma independente; uma falha de resolução (tarifa ou pedágio ausente) marca apenas aquela entrega como `pendente`, sem interromper o processamento das demais nem impedir a emissão do fechamento com o restante dos valores.

**Rationale**: FR-006 do spec.md desta HU. Bloquear o fechamento inteiro por uma configuração incompleta de uma única entrega reintroduziria o atraso manual que o projeto busca eliminar (SC-004).

**Alternatives considered**: Atribuir um valor padrão (ex. zero) à entrega sem tarifa foi descartado — mascararia um erro de configuração como se fosse um pagamento zero legítimo, um risco para a confiabilidade dos dados (princípio da constituição técnica, `mile_bag_audite/CONSTITUTION.md` §2.1).
