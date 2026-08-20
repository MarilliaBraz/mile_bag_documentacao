# Quickstart: HU13 — Emitir relatórios com identidade visual do tenant

**Feature**: [spec.md](./spec.md) | **Data model**: [data-model.md](./data-model.md) | **Contrato**: [contracts/identidade-visual-api.md](./contracts/identidade-visual-api.md)

## Pré-requisitos

- Um tenant já provisionado e com um administrador autenticado (ver [`../HU14-cadastrar-empresa-assinante/quickstart.md`](../HU14-cadastrar-empresa-assinante/quickstart.md)).
- Ao menos um fechamento já gerável para esse tenant (ver [`../HU12-gerar-fechamento-quinzenal/quickstart.md`](../HU12-gerar-fechamento-quinzenal/quickstart.md)).

## Passos

1. Como administrador do tenant, acessar a tela de configuração de identidade visual.
2. Enviar um logotipo válido (PNG/JPEG dentro do limite de tamanho) e preencher o cabeçalho.
3. Confirmar que `GET /api/v1/identidade-visual` retorna os dados enviados.
4. Gerar (ou reemitir) um relatório de fechamento para esse tenant.
5. Abrir o PDF gerado e conferir visualmente a presença do logotipo e do cabeçalho configurados.
6. Repetir os passos 1–5 com um segundo tenant, usando uma identidade visual diferente, e confirmar que os dois relatórios não se misturam (Acceptance Scenario 3).
7. Sem configurar identidade visual para um terceiro tenant, gerar um relatório e confirmar que ele sai com aparência padrão neutra, sem erro (Acceptance Scenario 4).

## Resultado esperado

- Cada relatório de fechamento reflete exclusivamente a identidade visual do próprio tenant (ou a aparência padrão, se não configurada).
- Alterar a identidade visual depois de um fechamento já emitido não altera o PDF anterior (Edge Case do spec).
