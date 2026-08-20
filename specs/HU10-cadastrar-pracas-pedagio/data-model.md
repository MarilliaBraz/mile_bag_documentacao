# Fase 1 — Modelo de Dados: HU10

**Feature**: [spec.md](./spec.md)

## Entidade: PracaPedagio

Representa RD08 (`../../requisitos/04-regras-dominio.md`).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `tenantId` | referência a Tenant | sim | Toda praça pertence a exatamente um tenant (RD01, RNF01) |
| `nome` | texto | sim | Identificação livre da praça (ex.: "Pedágio BR-060 km 12") |

## Entidade: PracaPedagioValor

Histórico de valores de uma praça, nunca sobrescrito (FR-002).

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | identificador | sim | — |
| `pracaPedagioId` | referência a PracaPedagio | sim | — |
| `valor` | decimal | sim | Valor unitário (um sentido) da praça |
| `vigenteDesde` | data | sim | MUST ser posterior à `vigenteDesde` de qualquer valor já cadastrado para a mesma praça (FR-005) |

**Regras de validação**:
- Não existe `vigenteAte` explícito: o valor vigente em uma data é sempre o de maior `vigenteDesde` que seja `<=` a essa data (research.md).
- O valor de reembolso de uma viagem é sempre `2 × valor` (ida + volta, RN06) — não modelado como campo, calculado em tempo de uso (FR-004).

**Relacionamentos**:
- `PracaPedagio` 1:N `PracaPedagioValor`.
- `PracaPedagio` N:1 `Tenant`.
- O snapshot de viagem (ver [spec-mãe](../001-milebag-user-stories/spec.md#key-entities)) grava o **valor resolvido** no momento do congelamento, não uma referência viva a `PracaPedagioValor` — por isso alterações futuras de valor não afetam viagens já congeladas (FR-003).
