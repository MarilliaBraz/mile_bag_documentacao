# HU12 — Gerar o fechamento quinzenal

**Persona:** Administrador do tenant / Back-office

**Como** administrador do tenant,
**eu quero** gerar o fechamento periódico de pagamento de entregas e de reembolso de pedágios,
**para que** eu saiba exatamente quanto devo pagar a cada motorista, sem apuração manual.

**Requisito(s) relacionado(s):** RF09

**Critérios de aceite:**
- O fechamento deve considerar os snapshots de rota congelados (HU04) e as tarifas vigentes no período (HU09, RD10).
- O relatório de pagamento de entregas e o de reembolso de pedágios devem ser gerados separadamente (RN08).
- O fechamento deve ser gerado sem exigir ajuste manual de valores.
