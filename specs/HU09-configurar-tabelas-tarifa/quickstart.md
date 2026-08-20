# Quickstart de validação: HU09 — Configurar tabelas de tarifa

**Pré-requisitos**:
- Tenant provisionado e administrador autenticado (ver quickstart de onboarding em [`../001-milebag-user-stories/`](../001-milebag-user-stories/)).
- Ao menos um motorista cadastrado no tenant, para validar tarifa por motorista.

## Passos

1. Autenticar como administrador do tenant.
2. Cadastrar uma tarifa `PADRAO` com `valorFixo` e `valorPorKm` — ver [contrato `POST /api/v1/tarifas`](./contracts/tarifas-api.md).
3. Cadastrar uma tarifa `MOTORISTA` para um motorista específico, com valores diferentes da padrão.
4. Cadastrar uma tarifa `REGIAO` para uma região específica.
5. Listar as tarifas do tenant (`GET /api/v1/tarifas`) e conferir que as três aparecem.
6. Autenticar como administrador de um segundo tenant e listar tarifas — confirmar que nenhuma das três aparece (isolamento, RNF01).

## Resultado esperado

- As três tarifas foram criadas sem erro (Acceptance Scenarios 1–3 do [spec.md](./spec.md)).
- A listagem do segundo tenant retorna vazia ou apenas tarifas próprias (Acceptance Scenario 4 / SC-003).
- Tentar cadastrar uma tarifa sem `valorFixo` nem `valorPorKm` retorna erro `400` (FR-005).

A resolução de qual tarifa se aplica a uma entrega real (motorista > região > padrão) é validada em conjunto com o fechamento — ver [quickstart de HU12](../HU12-gerar-fechamento-quinzenal/quickstart.md).
