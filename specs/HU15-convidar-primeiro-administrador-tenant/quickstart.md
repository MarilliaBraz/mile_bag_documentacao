# Quickstart: HU15 — Convidar o primeiro administrador do tenant

**Feature**: [spec.md](./spec.md) | **Data model**: [data-model.md](./data-model.md) | **Contrato**: [contracts/convite-api.md](./contracts/convite-api.md)

## Pré-requisitos

- Um tenant recém-cadastrado via HU14 (ver [`../HU14-cadastrar-empresa-assinante/quickstart.md`](../HU14-cadastrar-empresa-assinante/quickstart.md)), com e-mail de administrador indicado.

## Passos

1. Cadastrar um tenant (HU14) e confirmar, sem nenhuma ação adicional, que um convite com `status = PENDENTE` foi criado para o e-mail indicado.
2. Chamar `GET /api/v1/convites/{token}` com o token do convite recebido e confirmar `200` com os dados do tenant.
3. Chamar `POST /api/v1/convites/{token}/ativar` com credenciais válidas e confirmar `201` — a conta de administrador do tenant é criada.
4. Repetir o passo 3 com o mesmo token e confirmar que a segunda tentativa é recusada (`409`, Acceptance Scenario 3).
5. Gerar um convite, aguardar (ou simular) sua expiração, e confirmar que `GET`/ativação retornam `410` (Acceptance Scenario 4).
6. Cadastrar um segundo tenant, disparar `POST /api/v1/plataforma/tenants/{tenantId}/convites/reenviar` antes que o primeiro convite seja usado, e confirmar que o convite original passa a `INVALIDADO`.

## Resultado esperado

- Todo tenant cadastrado com sucesso recebe automaticamente um convite pendente.
- Um convite só pode ser usado uma vez, dentro do prazo de validade.
- Ao final da ativação, a conta criada só tem acesso ao tenant correspondente — validar com uma tentativa de acesso a outro tenant (mesmo teste de isolamento de [`../HU14-cadastrar-empresa-assinante/quickstart.md`](../HU14-cadastrar-empresa-assinante/quickstart.md)).
