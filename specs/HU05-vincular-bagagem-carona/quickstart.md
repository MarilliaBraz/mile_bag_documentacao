# Quickstart — Vincular bagagem carona à entrega

**Feature**: [spec.md](./spec.md) | **Modelo de dados**: [data-model.md](./data-model.md) | **Contratos**: [contracts/bagagem-carona-api.md](./contracts/bagagem-carona-api.md)

## Pré-requisitos

- Um tenant provisionado, com um motorista autenticado (ver quickstart da User Story 1 do spec-mãe).
- Uma entrega principal já autoatribuída e em andamento (HU01–HU02).
- Um segundo BDO capturado (da bagagem carona), ainda não vinculado a nenhuma entrega.

## Passos

1. Com a entrega principal em andamento, capturar o BDO da bagagem carona (mesmo fluxo de HU01).
2. Chamar `POST /api/v1/entregas/{entregaId}/bagagens-carona` com o `bdoId` da bagagem carona recém-capturada.
3. Confirmar que a resposta é `201 Created` e que `quilometragemComplementar` foi calculada.
4. Repetir o passo 2 com o mesmo `bdoId` — confirmar que a segunda tentativa retorna `409 Conflict` (FR-003).
5. Encerrar a entrega principal com comprovante de entrega (HU06).
6. Tentar vincular uma nova bagagem carona à mesma entrega — confirmar `409 Conflict` (FR-002).

## Resultado esperado

- A entrega principal lista a bagagem carona vinculada via `GET /api/v1/entregas/{entregaId}/bagagens-carona`.
- No fechamento do período (HU12), o valor pago pela bagagem carona reflete apenas `quilometragemComplementar`, nunca uma tarifa de viagem completa (SC-003).
