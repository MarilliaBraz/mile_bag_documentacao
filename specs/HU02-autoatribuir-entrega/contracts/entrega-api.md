# Contrato de API: Entrega — Autoatribuição (HU02)

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/entregas/{bdoId}/autoatribuir`

Atribui ao motorista autenticado a responsabilidade pela entrega referente ao BDO informado.

- **Request**: sem corpo — o motorista responsável é sempre o usuário autenticado (JWT).
- **Response 201**: Entrega criada/atualizada com `motoristaResponsavel` preenchido e `status = ATRIBUIDA`.
- **Erros**:
  - `404` — BDO não encontrado, não confirmado, ou não pertence ao tenant do usuário autenticado.
  - `409` — entrega já possui um motorista responsável (Edge Case de dupla atribuição).

## `GET /api/v1/entregas?motorista=me&status=EM_ANDAMENTO`

Lista as entregas autoatribuídas ao motorista autenticado que ainda estão em andamento (valor do enum `EntregaStatus`, ver `../data-model.md`).

- **Nota**: o valor `EM_ANDAMENTO` só passa a ser produzido depois que HU04 (congelamento da viagem) estiver implementada; até lá, este filtro não retorna resultados.
- **Response 200**: lista de entregas com dados resumidos do BDO associado.
