# API Contract: HU15 — Convidar o primeiro administrador do tenant

**Feature**: [../spec.md](../spec.md) | **Data model**: [../data-model.md](../data-model.md)

## Disparo automático (sem endpoint próprio)

O convite é criado e enviado automaticamente como efeito colateral de `POST /api/v1/plataforma/tenants` (contrato em [`../../HU14-cadastrar-empresa-assinante/contracts/tenants-api.md`](../../HU14-cadastrar-empresa-assinante/contracts/tenants-api.md)) — não há chamada separada para "enviar convite" no fluxo normal.

## `POST /api/v1/plataforma/tenants/{tenantId}/convites/reenviar`

Reenvia o convite de administrador para um tenant sem administrador ativo. Restrito ao administrador da plataforma.

- **Response 201**: novo convite criado; convite anterior pendente (se existir) passa a `INVALIDADO`.
- **Response 409**: tenant já possui um administrador ativo — reenvio não se aplica.
- **Response 403**: usuário autenticado não é administrador da plataforma.

## `GET /api/v1/convites/{token}`

Endpoint público (sem autenticação prévia), usado pela tela de ativação para validar o convite antes de exibir o formulário de credenciais.

- **Response 200**: `{ tenantNome: string, emailDestinatario: string, status: "PENDENTE" }`
- **Response 410**: convite expirado (`status = EXPIRADO`) — corpo orienta a solicitar reenvio.
- **Response 409**: convite já usado ou invalidado (`status = USADO` ou `INVALIDADO`).
- **Response 404**: token inexistente.

## `POST /api/v1/convites/{token}/ativar`

Ativa o convite e cria a conta de administrador do tenant.

- **Request**: `{ senha: string, ...demais credenciais exigidas pelo mecanismo de autenticação }`
- **Response 201**: conta de administrador criada; convite passa a `status = USADO`.
- **Response 410 / 409 / 404**: mesmas condições do `GET` acima — a ativação deve revalidar o estado do convite no momento do envio, não confiar apenas na checagem prévia do `GET`.
