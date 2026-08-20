# Quickstart de validação: HU11 — Parametrizar configurações do tenant

**Pré-requisitos**:
- Tenant provisionado e administrador autenticado.
- Um segundo tenant provisionado, para validar isolamento (SC-003).

## Passos

1. Cadastrar dois motivos de ocorrência — ver [contrato](./contracts/configuracoes-tenant-api.md).
2. Simular a leitura desses motivos como o app do motorista faria (`GET /api/v1/configuracoes/motivos-ocorrencia`) e conferir que ambos aparecem.
3. Desativar um dos motivos (`PATCH .../{id}` com `ativo: false`) e repetir a leitura — confirmar que só o motivo ativo aparece.
4. Marcar um campo do formulário de baixa como obrigatório (`PUT /api/v1/configuracoes/campos-baixa`).
5. Cadastrar uma base aeroportuária e uma companhia aérea atendida.
6. Alterar a periodicidade de fechamento para uma data futura e confirmar, via `GET`, que a periodicidade atual permanece a vigente até lá.
7. Repetir os passos 1, 4 e 5 no segundo tenant com valores diferentes, e confirmar (via `GET` autenticado em cada tenant) que nenhuma configuração vaza entre eles.

## Resultado esperado

- Motivo desativado deixa de aparecer para novos registros, sem ser excluído (Acceptance Scenario 1, FR-009).
- Campo obrigatório configurado é o que a interface de baixa do motorista deve exigir (Acceptance Scenario 2, SC-002 — validação de UI fica a cargo da implementação de `mile_bag_app`).
- Mudança de periodicidade não afeta o período em andamento (Acceptance Scenario 5, SC-004).
- Nenhuma configuração de um tenant aparece para o outro (Acceptance Scenario 6, SC-003).
