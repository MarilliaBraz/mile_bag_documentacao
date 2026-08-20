# Quickstart de Validação: HU02 — Autoatribuir a entrega

**Feature**: [spec.md](./spec.md) | **Contratos**: [contracts/entrega-api.md](./contracts/entrega-api.md) | **Modelo de dados**: [data-model.md](./data-model.md)

## Pré-requisitos

- HU01 validada: um BDO confirmado disponível (`statusConferencia = confirmado`), sem entrega atribuída.
- Dois usuários motoristas de teste, do mesmo tenant, para validar o cenário de dupla atribuição.

## Passos

1. Autenticar como Motorista A.
2. Autoatribuir a entrega do BDO confirmado (`POST /api/v1/entregas/{bdoId}/autoatribuir`).
3. Consultar a lista de entregas do Motorista A (`GET /api/v1/entregas?motorista=me`) e confirmar que a entrega aparece com `status = ATRIBUIDA` (o filtro `status=EM_ANDAMENTO` só passa a retornar resultados depois que HU04 estiver implementada).
4. Autenticar como Motorista B e tentar autoatribuir a mesma entrega.

## Resultado esperado

- Passo 2: retorna 201, com `motoristaResponsavel` = Motorista A.
- Passo 3: a entrega aparece na lista do Motorista A.
- Passo 4: retorna 409 — a entrega já está atribuída ao Motorista A; a lista do Motorista B não deve incluí-la.
