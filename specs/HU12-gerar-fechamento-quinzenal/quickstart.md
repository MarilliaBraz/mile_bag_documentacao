# Quickstart de validação: HU12 — Gerar o fechamento quinzenal

**Pré-requisitos**:
- Tenant provisionado, com tarifas ([HU09](../HU09-configurar-tabelas-tarifa/quickstart.md)) e praças de pedágio ([HU10](../HU10-cadastrar-pracas-pedagio/quickstart.md)) já cadastradas.
- Ao menos duas entregas concluídas com comprovante dentro do período a ser fechado, com snapshot de rota congelado (spec-mãe, HU04).
- Uma terceira entrega concluída sem tarifa aplicável cadastrada, para validar o cenário de pendência.

## Passos

1. Gerar o fechamento do período (`POST /api/v1/fechamentos`) — ver [contrato](./contracts/fechamento-api.md).
2. Conferir que o `status` retornado é `COM_PENDENCIAS` (por causa da terceira entrega) e que as outras duas aparecem com `statusItem = CALCULADO`.
3. Repetir a chamada de geração para o mesmo período e confirmar que retorna o mesmo fechamento (`200`), não cria um segundo (FR-005).
4. Baixar `relatorio-pagamento-entregas.pdf` e `relatorio-reembolso-pedagios.pdf` e conferir que são documentos separados, com os valores batendo com um cálculo manual de referência (quilometragem × tarifa; pedágio × 2).
5. Corrigir a configuração de tarifa da terceira entrega e, fora do escopo desta HU, verificar manualmente que o valor calculado nela bate com a tarifa aplicável.

## Resultado esperado

- Fechamento gerado sem ajuste manual, com totais corretos por motorista (SC-001).
- Segunda tentativa de geração não duplica o fechamento (SC-003).
- Entrega sem configuração aparece como pendente, sem travar o restante (FR-006, SC-002).
- Relatórios de entregas e de pedágios emitidos como documentos distintos (Acceptance Scenario 4, RN08).
