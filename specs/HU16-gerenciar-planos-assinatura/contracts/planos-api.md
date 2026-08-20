# API Contract: HU16 — Gerenciar planos de assinatura

**Feature**: [../spec.md](../spec.md) | **Data model**: [../data-model.md](../data-model.md)

Endpoints de CRUD de plano restritos ao administrador da plataforma. Consulta de plano vigente também acessível ao administrador do próprio tenant.

## `POST /api/v1/plataforma/planos`

- **Request**: `{ nome: string, limiteMotoristasAtivos: number, limiteVolumeMensalBdos: number }`
- **Response 201**: plano criado.
- **Response 400**: algum limite ≤ 0 (Edge Case 3).

## `GET /api/v1/plataforma/planos`

- **Response 200**: lista de planos, cada um com contagem de tenants associados (usada na UI antes de tentar remover).

## `DELETE /api/v1/plataforma/planos/{id}`

- **Response 204**: plano removido.
- **Response 409**: existe ao menos um tenant associado ao plano (FR-005, Acceptance Scenario 3).

## `PUT /api/v1/plataforma/tenants/{tenantId}/plano`

Troca o plano vigente de um tenant.

- **Request**: `{ planoId: string }`
- **Response 200**: `{ planoVigente: PlanoAssinatura, usoExcedeuLimiteNaTroca: boolean }` — o segundo campo aciona a sinalização de excedente (FR-006, Edge Case 1) quando `true`.
- **Response 404**: `planoId` inexistente.

## `GET /api/v1/plano`

Consulta o plano vigente do tenant autenticado. Sem segmento `tenants/me` no path — o tenant é sempre resolvido a partir do claim do token, nunca de um parâmetro do cliente, mesmo padrão usado nos contratos de HU09–HU12 (usado por HU14 e por qualquer HU que precise checar limites, ex. FR-017 do spec-mãe).

- **Response 200**: `{ planoVigente: PlanoAssinatura, usoAtual: { motoristasAtivos: number, bdosNoMesCorrente: number }, dentroDoLimite: boolean }`
