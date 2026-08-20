# Quickstart de Validação: HU03 — Calcular e escolher a rota da entrega

**Feature**: [spec.md](./spec.md) | **Contratos**: [contracts/rota-api.md](./contracts/rota-api.md) | **Modelo de dados**: [data-model.md](./data-model.md)

## Pré-requisitos

- HU02 validada: uma entrega autoatribuída ao motorista de teste.
- Serviço de rotas (Valhalla/OSRM) e geocodificação (Nominatim/Photon) disponíveis no ambiente (ver `../001-milebag-user-stories/plan.md`).
- Um endereço de entrega válido e um endereço propositalmente não geocodificável, para cobrir o Edge Case correspondente.

## Passos

1. Solicitar o cálculo de rota para a entrega de teste (`POST /api/v1/entregas/{entregaId}/rotas/calcular`).
2. Verificar que ao menos uma alternativa retorna com distância, tempo e pedágios.
3. Selecionar uma das alternativas sugeridas (`POST /api/v1/entregas/{entregaId}/rotas/selecionar`, `tipoSelecao = sugerida`).
4. Repetir o cálculo para uma entrega com endereço não geocodificável; ajustar manualmente o ponto de destino e recalcular.
5. Selecionar rota própria sem informar justificativa e confirmar que o sistema rejeita a seleção.
6. Repetir a seleção de rota própria informando a justificativa.

## Resultado esperado

- Passo 2: ao menos uma alternativa de rota é retornada.
- Passo 3: seleção registrada com sucesso, pronta para o congelamento em HU04.
- Passo 4: o recálculo funciona após o ajuste manual do destino.
- Passo 5: retorna 400 (justificativa ausente).
- Passo 6: seleção de rota própria registrada com sucesso.
