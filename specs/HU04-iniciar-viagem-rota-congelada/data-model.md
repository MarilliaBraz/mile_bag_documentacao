# Fase 1 — Modelo de Dados: HU04

**Feature**: [spec.md](./spec.md)

## Entidade: Viagem (snapshot de rota)

Registro imutável, criado no início da viagem a partir da Proposta de Rota selecionada (ver `../HU03-calcular-e-escolher-rota/data-model.md`). Ver `../../requisitos/04-regras-dominio.md` (RD04) e `../../requisitos/03-regras-negocio.md` (RN02).

| Campo | Descrição | Regra |
|---|---|---|
| id | Identificador único da viagem | — |
| entregaId | Entrega à qual a viagem pertence | Obrigatório; 1:1 — uma entrega tem no máximo uma viagem iniciada (FR-004) |
| kmIda | Quilometragem de ida, congelada no início | Imutável após a criação |
| kmVolta | Quilometragem de volta, congelada no início | Imutável após a criação |
| pracasPedagio | Lista de trechos com pedágio identificados na rota | Imutável; itens sem valor cadastrado são sinalizados (FR-006), não bloqueiam a criação |
| tempoEstimado | Tempo estimado do trajeto | Imutável após a criação |
| origemSelecao | `sugerida` \| `propria`, herdado da Proposta de Rota de HU03 | — |
| dataInicio | Data/hora do início da viagem | — |

**Validações**:
- Nenhum campo além dos de auditoria (ex. metadados técnicos) pode ser alterado após a criação do registro (FR-003).
- Só pode existir uma Viagem por Entrega (FR-004): tentativa de criar uma segunda MUST ser rejeitada.
- Toda Viagem deve ter uma Proposta de Rota selecionada como origem — não existe criação de Viagem "do zero".

**Efeito colateral na Entrega**: ao criar a Viagem (congelar a rota), esta HU é responsável por transicionar `Entrega.status` para `EntregaStatus.EM_ANDAMENTO` — enum canônico definido em `../HU02-autoatribuir-entrega/data-model.md`. Essa transição faz parte da mesma operação transacional de criação da Viagem (nunca uma etapa separada).

**Relacionamentos**: uma Viagem pertence a exatamente uma Entrega e é a base exclusiva para o cálculo do pagamento consumido no Fechamento (fora do escopo desta HU — ver spec guarda-chuva, User Story 2).
