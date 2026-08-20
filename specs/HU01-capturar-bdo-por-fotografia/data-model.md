# Fase 1 — Modelo de Dados: HU01

**Feature**: [spec.md](./spec.md)

## Entidade: BDO

Representa o documento de origem de uma entrega (telex WorldTracer impresso pela companhia aérea e fotografado pelo motorista). Ver `../../requisitos/04-regras-dominio.md` (RD02).

| Campo | Descrição | Regra |
|---|---|---|
| id | Identificador único do BDO | — |
| tenantId | Tenant proprietário do registro | Obrigatório; isolamento RNF01 |
| imagemOriginal | Referência à imagem fotografada | Obrigatória, preservada mesmo após conferência (FR-006) |
| camposExtraidos | Conjunto de campos extraídos por OCR (identificação do passageiro, endereço, voo/companhia aérea) | Preenchidos automaticamente; editáveis até a confirmação |
| statusConferencia | `pendente` \| `confirmado` | Só transita para `confirmado` por ação explícita do motorista (RN01, FR-005) |
| dataCaptura | Data/hora da fotografia | — |

**Validações**:
- Não pode existir BDO com `statusConferencia = confirmado` sem todos os campos obrigatórios preenchidos (extraídos ou corrigidos manualmente).
- `imagemOriginal` é imutável após a criação do registro.

**Relacionamentos**: um BDO confirmado é a origem de exatamente uma Entrega (ver `../HU02-autoatribuir-entrega/data-model.md` e o modelo consolidado em `../001-milebag-user-stories/data-model.md`, quando disponível).
