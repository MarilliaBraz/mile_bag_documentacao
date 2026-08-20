# Quickstart de validação: HU10 — Cadastrar praças de pedágio

**Pré-requisitos**:
- Tenant provisionado e administrador autenticado.

## Passos

1. Cadastrar uma praça de pedágio com valor inicial e data de vigência — ver [contrato `POST /api/v1/pracas-pedagio`](./contracts/pedagios-api.md).
2. Consultar `GET /api/v1/pracas-pedagio` e conferir o valor vigente exibido.
3. Cadastrar um novo valor para a mesma praça, com data de vigência posterior.
4. Consultar `GET /api/v1/pracas-pedagio/{id}/valores` e conferir que o valor anterior continua no histórico, não foi sobrescrito (FR-002).
5. Tentar cadastrar um novo valor com `vigenteDesde` anterior ao último cadastrado — confirmar rejeição (`400`, FR-005).
6. Consultar `GET /api/v1/pracas-pedagio/{id}/valor-em?data=...` com uma data anterior ao segundo valor — confirmar que retorna o primeiro valor, não o segundo.

## Resultado esperado

- Histórico de valores preservado após a segunda inserção (Acceptance Scenario 2 do [spec.md](./spec.md), SC-002).
- Vigência retroativa rejeitada (Edge Case 1).
- Consulta por data retorna sempre o valor correto para aquele ponto no tempo, base para o congelamento de rota (spec-mãe, FR-007) e para o reembolso em dobro no fechamento (FR-004) — validado em conjunto com [quickstart de HU12](../HU12-gerar-fechamento-quinzenal/quickstart.md).
