# Contrato de API: Viagem (HU04)

**Feature**: [../spec.md](../spec.md)

## `POST /api/v1/entregas/{entregaId}/viagem/iniciar`

Congela a rota previamente selecionada (`POST /api/v1/entregas/{entregaId}/rotas/selecionar`, contrato de HU03) em um snapshot imutável de viagem.

- **Request**: sem corpo — usa a última seleção de rota registrada para a entrega.
- **Response 201**: Viagem criada com `kmIda`, `kmVolta`, `pracasPedagio`, `tempoEstimado` e `dataInicio` gravados.
- **Erros**:
  - `404` — entrega não encontrada ou não pertence ao tenant.
  - `422` — entrega não possui rota selecionada (HU03 não concluída).
  - `409` — entrega já possui uma viagem iniciada.

## `GET /api/v1/entregas/{entregaId}/viagem`

Consulta o snapshot da viagem já iniciada, usado como base para o cálculo do pagamento (fora do escopo desta HU).

- **Response 200**: dados imutáveis do snapshot.
- **Erros**:
  - `404` — entrega sem viagem iniciada.

**Nota**: não existe endpoint de atualização (`PUT`/`PATCH`) para o snapshot de viagem — a imutabilidade (FR-003) é garantida pela ausência desse contrato, não apenas por regra de negócio.
