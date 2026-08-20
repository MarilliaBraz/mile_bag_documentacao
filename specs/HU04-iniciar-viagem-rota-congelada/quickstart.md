# Quickstart de Validação: HU04 — Iniciar a viagem com a rota congelada

**Feature**: [spec.md](./spec.md) | **Contratos**: [contracts/viagem-api.md](./contracts/viagem-api.md) | **Modelo de dados**: [data-model.md](./data-model.md)

## Pré-requisitos

- HU03 validada: uma entrega com rota selecionada (sugerida ou própria).
- Acesso para simular, no ambiente de teste, uma mudança posterior nos dados de mapa/pedágio (para validar a imutabilidade do snapshot).

## Passos

1. Iniciar a viagem para a entrega de teste (`POST /api/v1/entregas/{entregaId}/viagem/iniciar`).
2. Consultar o snapshot gravado (`GET /api/v1/entregas/{entregaId}/viagem`) e registrar os valores de `kmIda`, `kmVolta`, `pracasPedagio` e `tempoEstimado`.
3. Alterar, no ambiente de teste, o valor de uma praça de pedágio usada nessa rota (ou simular um novo resultado do serviço de mapas).
4. Consultar novamente o snapshot da mesma viagem.
5. Tentar iniciar uma segunda viagem para a mesma entrega.
6. Tentar iniciar uma viagem para uma entrega sem rota selecionada.

## Resultado esperado

- Passo 1: retorna 201 com o snapshot completo.
- Passo 4: os valores retornados são idênticos aos do passo 2, mesmo após a alteração do passo 3 — nenhuma mudança externa afeta o snapshot já gravado (SC-002 do spec desta HU).
- Passo 5: retorna 409.
- Passo 6: retorna 422.
