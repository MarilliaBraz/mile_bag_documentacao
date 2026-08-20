# Fase 1 — Modelo de Dados

**Feature**: [spec.md](./spec.md)

## Entidade: BagagemCarona

Representa uma bagagem adicional entregue na mesma viagem de uma entrega principal.

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | gerado pelo sistema |
| `bdoId` | referência ao BDO da bagagem carona | obrigatório; um BDO só pode originar uma bagagem carona (FR-003) |
| `entregaPrincipalId` | referência à Entrega | obrigatório; a entrega referenciada deve estar em andamento no momento da vinculação (FR-001, FR-002) |
| `quilometragemComplementar` | decimal (km) | obrigatório; calculado e congelado no momento da vinculação (RN03) — ver `research.md` |
| `criadoEm` | data/hora | obrigatório |

**Relacionamentos**:
- `BagagemCarona` N:1 `Entrega` (entrega principal) — uma entrega pode ter zero ou mais bagagens carona (FR-001).
- `BagagemCarona` 1:1 `BDO` — o BDO de origem da bagagem carona.

**Regras de validação**:
- Rejeitar a criação se `entregaPrincipal.status` já for `EntregaStatus.ENCERRADA_COM_SUCESSO` (FR-002, RD05 em `../../requisitos/04-regras-dominio.md`).
- Rejeitar a criação se o `bdoId` já estiver vinculado a outra `BagagemCarona` (FR-003).
- Ao reabrir a entrega principal por ocorrência (HU07), o vínculo de `BagagemCarona` existente não é alterado nem removido (FR-005).

## Entidade: Entrega (referenciada, não redefinida aqui)

Ver o modelo completo, incluindo o enum canônico `EntregaStatus`, em `../HU02-autoatribuir-entrega/data-model.md`. Para esta HU, basta que a entrega principal exponha seu `status` (`EntregaStatus.EM_ANDAMENTO` / `ENCERRADA_COM_SUCESSO` / `REABERTA`) para validar as regras acima.
