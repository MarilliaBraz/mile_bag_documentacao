# Contrato de API: BDO (HU01)

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/bdos`

Envia a fotografia do BDO e dispara a extração automática de campos.

- **Request**: multipart/form-data com a imagem capturada.
- **Response 201**: BDO criado com `statusConferencia = pendente` e `camposExtraidos` preenchidos pelo OCR (alguns podem vir vazios, caso o OCR não os reconheça).
- **Erros**:
  - `400` — imagem ausente ou em formato não suportado.
  - `401` — motorista não autenticado.

## `PATCH /api/v1/bdos/{id}/conferencia`

Confirma (com eventuais correções) os campos extraídos, encerrando a etapa de conferência obrigatória.

- **Request**: corpo com os valores finais de `camposExtraidos` (iguais ou corrigidos em relação aos extraídos).
- **Response 200**: BDO com `statusConferencia = confirmado`.
- **Erros**:
  - `400` — algum campo obrigatório ausente.
  - `404` — BDO não encontrado (ou não pertence ao tenant do usuário autenticado — RNF01).
  - `409` — BDO já confirmado anteriormente.

## `GET /api/v1/bdos?possivelDuplicidade=true&campoChave={valor}`

Consulta usada pelo front-end para sinalizar possível duplicidade antes da confirmação (Edge Case do spec).

- **Response 200**: lista de BDOs existentes com o mesmo campo-chave, para o tenant do usuário autenticado.
