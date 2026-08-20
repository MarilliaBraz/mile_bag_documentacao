# API Contract: HU13 — Identidade visual do tenant

**Feature**: [../spec.md](../spec.md) | **Data model**: [../data-model.md](../data-model.md)

Todos os endpoints exigem autenticação (token com claim de tenant) e são restritos ao perfil de administrador do próprio tenant (RD12). O `tenantId` nunca é recebido como parâmetro — é resolvido a partir do token, conforme o padrão de isolamento descrito no plano-mãe.

## `GET /api/v1/identidade-visual`

Retorna a identidade visual configurada para o tenant autenticado. Sem segmento `tenants/me` no path — o tenant é sempre resolvido a partir do claim do token, nunca de um parâmetro do cliente, mesmo padrão usado nos contratos de HU09–HU12.

- **Response 200**: `{ logotipoUrl: string | null, cabecalho: string | null, atualizadoEm: string }`
- **Response 200 (sem configuração)**: `{ logotipoUrl: null, cabecalho: null, atualizadoEm: null }` — aparência padrão neutra será usada (FR-004).

## `PUT /api/v1/identidade-visual`

Cria ou substitui a identidade visual do tenant autenticado.

- **Request**: multipart/form-data — `logotipo` (arquivo de imagem, opcional), `cabecalho` (texto, opcional).
- **Response 200**: identidade visual atualizada, mesmo formato do GET.
- **Response 400**: arquivo de logotipo em formato ou tamanho não suportado — mensagem indicando o motivo, identidade anterior preservada (Edge Case 1 do spec).
- **Response 403**: usuário autenticado não é administrador do tenant.

## Dependência entre features (sem endpoint próprio nesta HU)

Esta HU não define endpoint para "buscar o relatório" — isso já existe em HU12. Os dois endpoints de relatório já definidos em [`../../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md`](../../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md):

- `GET /api/v1/fechamentos/{id}/relatorio-pagamento-entregas.pdf`
- `GET /api/v1/fechamentos/{id}/relatorio-reembolso-pedagios.pdf`

passam a consultar a `IdentidadeVisualTenant` (definida nesta HU) internamente ao montar o PDF, assim que ela existir para o tenant. Não há mudança de contrato nesses dois endpoints — só no conteúdo visual do arquivo retornado. Ver nota equivalente em [`../../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md`](../../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md).
