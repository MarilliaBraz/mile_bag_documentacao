# Fase 1 — Modelo de Dados: HU09

**Feature**: [spec.md](./spec.md)

## Entidade: TabelaTarifa

Representa RD09 (`../../requisitos/04-regras-dominio.md`).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | Toda tarifa pertence a exatamente um tenant (RD01, RNF01) |
| `escopo` | enum: `PADRAO`, `MOTORISTA`, `REGIAO` | sim | Define a especificidade da tarifa (FR-004) |
| `motoristaId` | referência a Usuário (perfil motorista) | condicional | Obrigatório e único por tenant quando `escopo = MOTORISTA`; nulo nos demais casos |
| `regiao` | texto | condicional | Obrigatório quando `escopo = REGIAO`; nulo nos demais casos |
| `valorFixo` | decimal | condicional | Ao menos um entre `valorFixo` e `valorPorKm` é obrigatório (FR-005) |
| `valorPorKm` | decimal | condicional | Idem acima |
| `vigenteDesde` | data | sim | Data a partir da qual a tarifa passa a ser aplicada nos fechamentos |

**Regras de validação**:
- Um tenant tem no máximo uma tarifa com `escopo = PADRAO` vigente por vez (Assumption do spec.md).
- Um tenant tem no máximo uma tarifa `MOTORISTA` vigente por motorista, e no máximo uma tarifa `REGIAO` vigente por valor de região (evita ambiguidade na resolução).

**Relacionamentos**:
- `TabelaTarifa` N:1 `Tenant`.
- `TabelaTarifa` N:1 `Usuário` (quando `escopo = MOTORISTA`).
- `TabelaTarifa` é consumida por `Fechamento` (ver [HU12/data-model.md](../HU12-gerar-fechamento-quinzenal/data-model.md)) no momento da apuração — não há referência de volta.

**Não modificado nesta HU**: `Tenant`, `Usuário` — ver [data-model do spec-mãe](../001-milebag-user-stories/spec.md#key-entities) para a definição completa dessas entidades.
