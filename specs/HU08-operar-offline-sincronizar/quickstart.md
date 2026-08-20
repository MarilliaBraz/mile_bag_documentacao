# Quickstart — Operar offline e sincronizar automaticamente

**Feature**: [spec.md](./spec.md) | **Modelo de dados**: [data-model.md](./data-model.md) | **Contratos**: [contracts/sincronizacao-api.md](./contracts/sincronizacao-api.md)

## Pré-requisitos

- Um motorista autenticado na PWA, com ao menos uma entrega disponível para captura.
- Capacidade de simular perda de conectividade no dispositivo/navegador de teste (ex.: modo avião, ou `context.setOffline()` em teste automatizado — ver `../001-milebag-user-stories/research.md`, seção 1).

## Passos

1. Colocar o dispositivo em modo offline.
2. Executar, nessa condição, o ciclo: capturar um BDO (HU01) → autoatribuir (HU02) → escolher rota (HU03) → iniciar viagem (HU04) → registrar comprovante (HU06). Confirmar que cada ação é aceita localmente e aparece na interface como "pendente de sincronização" (FR-002, FR-003).
3. Restaurar a conectividade do dispositivo.
4. Confirmar que a aplicação dispara `POST /api/v1/sync/eventos` automaticamente, sem ação do motorista (FR-004).
5. Verificar, na resposta, que todos os itens do lote retornam `resultado: "ACEITO"` e que a ordem de processamento respeitou `sequenciaLocal`.
6. Reenviar deliberadamente o mesmo lote (simulando uma reconexão que caiu antes da confirmação chegar ao cliente) — confirmar que os itens agora retornam `resultado: "DUPLICADO"`, sem criar registros repetidos (FR-006).
7. Consultar `GET /api/v1/sync/status?idsLocais=...` com os `idLocal` do passo 2 — confirmar que todos aparecem como `ACEITO`.

## Resultado esperado

- Todos os registros criados offline chegam ao servidor após a reconexão, na ordem correta, sem exigir ação manual (SC-001, SC-002, SC-004).
- Nenhum registro é duplicado, mesmo com reenvio do mesmo lote (SC-003).
