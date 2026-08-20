# Contrato de API: Praças de Pedágio (HU10)

**Autenticação**: token com claim de tenant; perfil exigido: administrador do tenant (leitura também permitida à aplicação do motorista, para exibir pedágios na escolha de rota — HU03 do spec-mãe).

## `POST /api/v1/pracas-pedagio`

Cadastra uma nova praça de pedágio, já com seu primeiro valor.

- **Request**: `nome`, `valor`, `vigenteDesde`.
- **Response 201**: praça criada, com `id`.

## `GET /api/v1/pracas-pedagio`

Lista as praças de pedágio do tenant autenticado, com o valor atualmente vigente de cada uma.

- **Response 200**: lista de praças (`id`, `nome`, `valorVigente`, `vigenteDesde`).

## `GET /api/v1/pracas-pedagio/{id}/valores`

Lista o histórico completo de valores de uma praça.

- **Response 200**: lista de `{ valor, vigenteDesde }`, ordenada por `vigenteDesde` decrescente.

## `POST /api/v1/pracas-pedagio/{id}/valores`

Cadastra um novo valor de vigência para uma praça já existente.

- **Request**: `valor`, `vigenteDesde`.
- **Response 201**: novo valor criado.
- **Erros**:
  - `400` — `vigenteDesde` anterior ou igual à `vigenteDesde` do valor vigente mais recente (FR-005).
  - `404` — praça inexistente ou pertencente a outro tenant.

## `GET /api/v1/pracas-pedagio/{id}/valor-em?data=AAAA-MM-DD`

Consulta o valor vigente de uma praça em uma data específica — usado pelo serviço de congelamento de rota (spec-mãe, FR-007) e pelo fechamento (HU12).

- **Response 200**: `{ valor, vigenteDesde }`.
- **Erros**: `404` — praça sem nenhum valor vigente até a data informada.
