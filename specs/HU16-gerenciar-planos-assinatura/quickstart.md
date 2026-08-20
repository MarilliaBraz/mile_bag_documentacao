# Quickstart: HU16 — Gerenciar planos de assinatura

**Feature**: [spec.md](./spec.md) | **Data model**: [data-model.md](./data-model.md) | **Contrato**: [contracts/planos-api.md](./contracts/planos-api.md)

## Pré-requisitos

- Um usuário com perfil de administrador da plataforma autenticado.

## Passos

1. Chamar `POST /api/v1/plataforma/planos` com limites positivos e confirmar `201`.
2. Tentar cadastrar um plano com `limiteMotoristasAtivos: 0` e confirmar `400` (Edge Case 3).
3. Usar o plano criado para cadastrar um tenant (ver [`../HU14-cadastrar-empresa-assinante/quickstart.md`](../HU14-cadastrar-empresa-assinante/quickstart.md)).
4. Tentar `DELETE /api/v1/plataforma/planos/{id}` desse plano e confirmar `409`, já que há um tenant associado (Acceptance Scenario 3).
5. Cadastrar um segundo plano e chamar `PUT /api/v1/plataforma/tenants/{tenantId}/plano` para migrar o tenant do passo 3 para ele; confirmar `200` com `planoVigente` atualizado.
6. Chamar `GET /api/v1/plano` autenticado como o tenant migrado e confirmar que reflete apenas o novo plano (Acceptance Scenario 4 — nunca mais de um plano vigente simultâneo).

## Resultado esperado

- Plano com tenants associados nunca é removido.
- Troca de plano é imediata, sem qualquer etapa de reimplantação.
- `usoExcedeuLimiteNaTroca` sinaliza corretamente quando o tenant migrado já excede os limites do novo plano, sem bloquear a troca.
