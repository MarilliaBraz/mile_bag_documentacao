# Contrato de API: Fechamento (HU12)

**Autenticação**: token com claim de tenant; perfil exigido: administrador do tenant.

## `POST /api/v1/fechamentos`

Gera o fechamento de um período. Idempotente por `(tenant, período)` — ver [research.md](./research.md).

- **Request**: `periodoInicio`, `periodoFim`.
- **Response 201**: fechamento criado, com `status` (`COMPLETO` ou `COM_PENDENCIAS`) e resumo de totais por motorista.
- **Response 200**: se já existir um fechamento para o mesmo `(tenant, período)`, retorna o existente em vez de recriar (FR-005).
- **Erros**: `409` — período informado se sobrepõe parcialmente a um fechamento já existente (sem coincidir exatamente).

## `GET /api/v1/fechamentos`

Lista os fechamentos já gerados pelo tenant autenticado (FR-008 — nunca de outro tenant).

- **Query params opcionais**: `de`, `ate` (filtro por data).
- **Response 200**: lista de fechamentos, com `status` e totais gerais.

## `GET /api/v1/fechamentos/{id}`

- **Response 200**: detalhe do fechamento, com lista de `ItemFechamento` (por motorista e entrega), incluindo itens `PENDENTE` com `motivoPendencia` (FR-006).

## `GET /api/v1/fechamentos/{id}/relatorio-pagamento-entregas.pdf`

Relatório de pagamento de entregas, separado do de pedágios (FR-003, RN08).

- **Response 200**: `application/pdf`.

## `GET /api/v1/fechamentos/{id}/relatorio-reembolso-pedagios.pdf`

- **Response 200**: `application/pdf`.

**Nota**: a identidade visual aplicada a esses dois PDFs é responsabilidade de HU13 (fora desta pasta) — este contrato não define o layout, apenas a existência dos dois relatórios como documentos distintos.
