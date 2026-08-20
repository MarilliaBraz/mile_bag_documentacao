# Fase 0 — Pesquisa e Decisões Técnicas: HU11

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Nenhuma decisão de tecnologia nova além do que já está registrado em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção **"3. Isolamento multi-tenant"** — as cinco configurações desta HU são apenas mais entidades sujeitas à mesma política de RLS por coluna de tenant.

## Desativação em vez de exclusão de motivo de ocorrência

**Decision**: Motivos de ocorrência têm um campo `ativo` (booleano); remover um motivo desativa-o, nunca exclui a linha.

**Rationale**: Ocorrências já registradas referenciam o motivo por chave estrangeira (RD07, `../../requisitos/04-regras-dominio.md`); excluir a linha quebraria a integridade referencial e o histórico de ocorrências passadas (RN05, preservação de histórico). Desativação é o padrão mínimo suficiente — não há requisito de auditoria mais sofisticado (ex. soft-delete com trilha de quem desativou) documentado nas evidências.

**Alternatives considered**: Exclusão física com preservação do texto do motivo copiado para dentro do registro de ocorrência (denormalização) foi descartada por adicionar complexidade sem necessidade — desativação resolve o requisito com uma coluna a mais.

## Troca de periodicidade de fechamento no meio de um período

**Decision**: A configuração de periodicidade tem, como as tarifas e os pedágios, um histórico com data de vigência; o fechamento em andamento usa a periodicidade vigente no início desse período, não a mais recente.

**Rationale**: Mesma lógica já aplicada a tarifas (HU09) e pedágios (HU10) — nunca alterar retroativamente um cálculo já em curso ou já emitido (RN02, por analogia). Reaproveita um padrão já validado no projeto em vez de introduzir um mecanismo novo.
