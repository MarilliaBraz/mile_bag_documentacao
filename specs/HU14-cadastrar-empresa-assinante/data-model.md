# Data Model: HU14 — Cadastrar empresa assinante (tenant)

**Feature**: [spec.md](./spec.md)

## Entidade: Tenant

Empresa terceirizada de entrega, assinante do sistema (RD01).

| Campo | Tipo | Regras |
|---|---|---|
| `id` | identificador único | Gerado pelo sistema |
| `identificadorEmpresa` | texto (ex.: CNPJ) | Obrigatório; único em todo o sistema (FR-003) |
| `nome` | texto | Obrigatório |
| `planoId` | referência (PlanoAssinatura) | Obrigatório — um tenant sem plano não pode ser criado (FR-002) |
| `criadoEm` | data/hora | Obrigatório |
| `status` | enumeração (`ativo`) | Definido na criação; ver [`../HU16-gerenciar-planos-assinatura/data-model.md`](../HU16-gerenciar-planos-assinatura/data-model.md) para estados relacionados a limite de plano |

**Relações**: 1 Tenant → 1 PlanoAssinatura vigente (RD11). 1 Tenant → 0..1 Usuário administrador (associado por HU15, fora do escopo desta HU).

**Regras de validação**:
- `identificadorEmpresa` é único — tentativa de duplicidade é rejeitada (FR-003, Edge Case 1).
- Criação de `Tenant` e associação a `planoId` ocorrem na mesma operação atômica (FR-006).
- Toda entidade de domínio criada posteriormente para este tenant (motoristas, BDOs, tarifas etc.) referencia `Tenant.id` como discriminador, sujeito a Row Level Security (FR-004).
