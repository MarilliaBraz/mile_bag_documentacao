# Data Model: HU13 — Emitir relatórios com identidade visual do tenant

**Feature**: [spec.md](./spec.md)

## Entidade: IdentidadeVisualTenant

Configuração de aparência de um tenant, usada na emissão de relatórios.

| Campo | Tipo | Regras |
|---|---|---|
| `tenantId` | referência (Tenant) | Obrigatório; único — um tenant tem no máximo uma identidade visual vigente (RI07) |
| `logotipo` | arquivo de imagem | Opcional; quando ausente, relatórios usam aparência padrão neutra (FR-004) |
| `cabecalho` | texto | Opcional; nome/dados exibidos no topo do relatório |
| `atualizadoEm` | data/hora | Obrigatório; registra quando a identidade foi configurada/alterada pela última vez |

**Relações**: 1 Tenant → 0..1 IdentidadeVisualTenant.

**Regras de validação**:
- `tenantId` nunca é alterável após a criação do registro (RN — isolamento entre tenants, FR-003).
- Atualizar `logotipo` ou `cabecalho` não deve reprocessar relatórios já emitidos anteriormente (FR-005) — a identidade é lida no momento da geração de cada relatório, não vinculada retroativamente aos já gerados.

## Nota: sem entidade própria para o relatório

Esta HU não cria (nem deveria criar) uma entidade persistida para "o relatório" — o PDF é uma **renderização** de `Fechamento` (entidade definida em [`../HU12-gerar-fechamento-quinzenal/data-model.md`](../HU12-gerar-fechamento-quinzenal/data-model.md), junto com `ItemFechamento`) combinada com `IdentidadeVisualTenant` (definida acima), aplicada no momento da geração de cada arquivo.

Fluxo: ao gerar o PDF de um `Fechamento` (endpoints já definidos em [`../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md`](../HU12-gerar-fechamento-quinzenal/contracts/fechamento-api.md)), o gerador de relatório lê a `IdentidadeVisualTenant` vigente para o `tenantId` do fechamento e a aplica ao layout (logotipo/cabeçalho). Não há busca, cópia ou entidade intermediária — apenas leitura em tempo de renderização.
