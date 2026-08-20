# Data Model: HU16 — Gerenciar planos de assinatura

**Feature**: [spec.md](./spec.md)

## Entidade: PlanoAssinatura

Plano comercial da plataforma, com limites operacionais (RD11).

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | Gerado pelo sistema |
| `nome` | texto | Obrigatório; não precisa ser único (Edge Case 2) |
| `limiteMotoristasAtivos` | número inteiro | Obrigatório; MUST ser > 0 (Edge Case 3) |
| `limiteVolumeMensalBdos` | número inteiro | Obrigatório; MUST ser > 0 (Edge Case 3) |
| `criadoEm` | data/hora | Obrigatório |

**Relações**: 1 PlanoAssinatura → 0..N Tenant (associados no presente). Remoção bloqueada enquanto essa contagem for > 0 (FR-005).

## Entidade: HistoricoPlanoTenant

Registro de qual plano vigorou para um tenant em cada intervalo de tempo — suporta rastreabilidade (Constitution Check) e a sinalização de excedente ao trocar de plano.

| Campo | Tipo | Regras |
|---|---|---|
| `tenantId` | referência (Tenant) | Obrigatório |
| `planoId` | referência (PlanoAssinatura) | Obrigatório |
| `vigenteDesde` | data/hora | Obrigatório |
| `vigenteAte` | data/hora | Nulo enquanto for o plano vigente; preenchido quando substituído |
| `usoExcedeuLimiteNaTroca` | booleano | Marcado quando, no momento da troca, o uso atual do tenant já excedia os limites do novo plano (FR-006) |

**Regras de validação**:
- Para um `tenantId`, no máximo um registro tem `vigenteAte = null` por vez — é o plano vigente (FR-003, Acceptance Scenario 4).
- Trocar o plano de um tenant fecha o registro vigente (`vigenteAte = agora`) e cria um novo com `vigenteDesde = agora` na mesma operação (FR-004).
