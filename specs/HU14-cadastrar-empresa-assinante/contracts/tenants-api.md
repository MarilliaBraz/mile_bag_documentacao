# API Contract: HU14 — Cadastrar empresa assinante (tenant)

**Feature**: [../spec.md](../spec.md) | **Data model**: [../data-model.md](../data-model.md)

Endpoints restritos ao perfil de administrador da plataforma (provedor) — não confundir com administrador de um tenant (RD12).

## `POST /api/v1/plataforma/tenants`

Cadastra uma nova empresa assinante.

- **Request**: `{ identificadorEmpresa: string, nome: string, planoId: string }`
- **Response 201**: `{ id: string, identificadorEmpresa: string, nome: string, planoId: string, status: "ativo", criadoEm: string }`
- **Response 400**: `planoId` ausente ou inválido (FR-002).
- **Response 409**: `identificadorEmpresa` já cadastrado em outro tenant (FR-003, Edge Case 1).
- **Response 403**: usuário autenticado não é administrador da plataforma.

Falhas durante o processamento (ex.: erro ao persistir) não deixam registro parcial — a operação é tudo-ou-nada (FR-006); o cliente deve tratar qualquer resposta diferente de 201 como "tenant não criado".

## `GET /api/v1/plataforma/tenants/{id}`

Consulta um tenant cadastrado, usada para confirmar que nenhuma pendência de infraestrutura ficou em aberto (Acceptance Scenario 3).

- **Response 200**: mesmo formato do `POST`, sem campo de pendência de implantação (porque não existe tal etapa — FR-005).
- **Response 404**: tenant inexistente.
