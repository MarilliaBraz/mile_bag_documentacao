# Contrato de API: Tarifas (HU09)

**Autenticação**: token com claim de tenant; perfil exigido: administrador do tenant.

## `POST /api/v1/tarifas`

Cadastra uma nova tarifa (padrão, por motorista ou por região).

- **Request**: `escopo` (`PADRAO`|`MOTORISTA`|`REGIAO`), `motoristaId` (se `MOTORISTA`), `regiao` (se `REGIAO`), `valorFixo`, `valorPorKm`, `vigenteDesde`.
- **Response 201**: tarifa criada, com `id`.
- **Erros**:
  - `400` — nem `valorFixo` nem `valorPorKm` informados (FR-005).
  - `400` — `escopo = MOTORISTA` sem `motoristaId`, ou `escopo = REGIAO` sem `regiao`.
  - `409` — já existe tarifa vigente com o mesmo escopo/motorista/região para o tenant.

## `GET /api/v1/tarifas`

Lista as tarifas do tenant autenticado (FR-006 — nunca retorna tarifas de outro tenant).

- **Query params opcionais**: `escopo`, `motoristaId`, `regiao`.
- **Response 200**: lista de tarifas.

## `GET /api/v1/tarifas/{id}`

- **Response 200**: detalhe da tarifa.
- **Erros**: `404` — tarifa inexistente ou pertencente a outro tenant (não deve vazar a existência de registros de outros tenants).

## `PUT /api/v1/tarifas/{id}`

Atualiza valores ou vigência de uma tarifa existente.

- **Request**: mesmos campos de valor de `POST`.
- **Response 200**: tarifa atualizada.
- **Nota**: não recalcula viagens já congeladas nem fechamentos já emitidos (Edge Case do spec.md) — afeta apenas apurações futuras.

## `DELETE /api/v1/tarifas/{id}`

Remove uma tarifa (ex.: motorista desligado, região descontinuada).

- **Response 204**.
- **Erros**: `409` — tarifa referenciada por um fechamento já emitido; nesse caso, encerrar a vigência (`vigenteAte`) em vez de excluir é a alternativa recomendada, não coberta nesta HU.
