# Data Model: HU15 — Convidar o primeiro administrador do tenant

**Feature**: [spec.md](./spec.md)

## Entidade: ConviteAdministrador

Convite de ativação de conta, associado a um tenant recém-cadastrado.

| Campo | Tipo | Regras |
|---|---|---|
| `id` / `token` | identificador único opaco | Gerado aleatoriamente; não previsível (ver research.md) |
| `tenantId` | referência (Tenant) | Obrigatório |
| `emailDestinatario` | texto | Obrigatório; validado como e-mail bem formado |
| `status` | enumeração `ConviteStatus` (`PENDENTE`, `USADO`, `EXPIRADO`, `INVALIDADO`) | Obrigatório; transições descritas abaixo |
| `criadoEm` | data/hora | Obrigatório |
| `expiraEm` | data/hora | Obrigatório; calculado a partir de `criadoEm` + prazo de validade configurado |
| `usadoEm` | data/hora | Preenchido apenas quando `status = USADO` |

**Relações**: 1 Tenant → 0..N ConviteAdministrador (histórico), mas no máximo 1 com `status = PENDENTE` por vez (FR-006).

**Transições de estado** (`ConviteStatus`, SCREAMING_SNAKE_CASE por `mile_bag_audite/CONVENTIONS.md` §4.1 — mesma convenção do `EntregaStatus` canônico em `../HU02-autoatribuir-entrega/data-model.md`):

```
PENDENTE --(destinatário ativa dentro do prazo)--> USADO
PENDENTE --(prazo expira sem uso)--> EXPIRADO
PENDENTE --(novo convite emitido para o mesmo tenant)--> INVALIDADO
```

**Regras de validação**:
- Só um convite com `status = PENDENTE` pode existir por tenant; emitir um novo convite muda o anterior para `INVALIDADO` (FR-006, Edge Case 3).
- Um convite com `status` diferente de `PENDENTE` nunca pode ser ativado (FR-004, Acceptance Scenario 3/4).
- Ativar um convite (`PENDENTE → USADO`) cria a conta de Usuário associada a `tenantId` com perfil de administrador do tenant (FR-003) — nunca com acesso a outro tenant.
