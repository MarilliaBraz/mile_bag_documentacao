# Quickstart: HU14 — Cadastrar empresa assinante (tenant)

**Feature**: [spec.md](./spec.md) | **Data model**: [data-model.md](./data-model.md) | **Contrato**: [contracts/tenants-api.md](./contracts/tenants-api.md)

## Pré-requisitos

- Um usuário com perfil de administrador da plataforma autenticado.
- Ao menos um plano de assinatura já cadastrado (ver [`../HU16-gerenciar-planos-assinatura/quickstart.md`](../HU16-gerenciar-planos-assinatura/quickstart.md)).

## Passos

1. Como administrador da plataforma, chamar `POST /api/v1/plataforma/tenants` com os dados de uma nova empresa e um `planoId` válido.
2. Confirmar resposta `201` com o tenant criado e `status: "ativo"`.
3. Repetir o cadastro com o mesmo `identificadorEmpresa` e confirmar resposta `409` (Edge Case 1).
4. Tentar cadastrar um novo tenant sem `planoId` e confirmar resposta `400` (Acceptance Scenario 4).
5. Autenticado como usuário de outro tenant já existente, tentar acessar dados do tenant recém-criado e confirmar que o acesso é negado (Acceptance Scenario 2 / isolamento).
6. Consultar `GET /api/v1/plataforma/tenants/{id}` do novo tenant e confirmar que nenhum campo indica pendência de implantação.

## Resultado esperado

- Tenant criado em uma única operação, sem etapa de infraestrutura.
- Isolamento de dados válido desde a criação.
- Próximo passo do fluxo de onboarding: convidar o primeiro administrador (ver [`../HU15-convidar-primeiro-administrador-tenant/quickstart.md`](../HU15-convidar-primeiro-administrador-tenant/quickstart.md)).
