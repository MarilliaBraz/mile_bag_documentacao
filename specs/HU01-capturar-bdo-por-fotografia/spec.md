# Feature Specification: HU01 — Capturar o BDO por fotografia

**Feature Branch**: `HU01-capturar-bdo-por-fotografia`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU de origem: `../../HUs/motorista/HU01-capturar-bdo-por-fotografia.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF01), `../../requisitos/03-regras-negocio.md` (RN01), `../../requisitos/04-regras-dominio.md` (RD02), `../../requisitos/05-regras-interface.md` (RI01). Spec guarda-chuva: `../001-milebag-user-stories/spec.md`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Capturar o BDO por fotografia (Priority: P1)

Como motorista, eu quero fotografar o BDO recebido da companhia aérea, para que os dados da entrega sejam registrados no sistema sem que eu precise digitá-los.

**Why this priority**: É o ponto de entrada de toda entrega — nenhuma das demais HUs (autoatribuição, rota, viagem, comprovante) existe sem um BDO capturado antes. Substitui a digitação manual, causa raiz do retrabalho e erro descritos no problema do projeto.

**Independent Test**: Pode ser testada isoladamente fotografando um BDO real e verificando que os campos extraídos aparecem corretamente na tela de conferência, sem depender de nenhuma outra HU.

**Acceptance Scenarios**:

1. **Given** que o motorista está autenticado e tem a câmera disponível, **When** ele fotografa um BDO, **Then** o sistema extrai automaticamente os campos do padrão WorldTracer e exibe uma tela de conferência com os valores sugeridos.
2. **Given** que a tela de conferência está exibindo os valores extraídos, **When** o motorista revisa e, se necessário, corrige algum campo, **Then** o registro só é salvo após a confirmação explícita do motorista — nunca automaticamente a partir do OCR.
3. **Given** que o BDO foi confirmado, **When** o registro é salvo, **Then** a imagem original fotografada fica anexada ao registro como prova documental, disponível para consulta posterior.

### Edge Cases

- O que acontece quando o OCR não consegue extrair um ou mais campos obrigatórios? O motorista deve poder preencher manualmente os campos faltantes na própria tela de conferência antes de confirmar.
- O que acontece quando a foto está ilegível (desfocada, mal enquadrada)? O sistema deve permitir refazer a foto antes de prosseguir para a extração.
- O que acontece quando o motorista fotografa um BDO já capturado anteriormente (duplicidade)? O sistema deve sinalizar a possível duplicidade ao motorista antes de confirmar um novo registro.
- O que acontece sem conexão com a internet? A captura e a tela de conferência devem funcionar normalmente, com o registro pendente de sincronização (tratado em HU08).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir ao motorista capturar uma fotografia do BDO a partir do dispositivo.
- **FR-002**: O sistema MUST extrair automaticamente os campos do padrão WorldTracer a partir da fotografia capturada.
- **FR-003**: O sistema MUST apresentar uma tela de conferência com os valores extraídos antes de qualquer gravação definitiva do registro.
- **FR-004**: O sistema MUST permitir a correção manual de qualquer campo extraído automaticamente, na tela de conferência.
- **FR-005**: O sistema MUST impedir a gravação do registro sem confirmação explícita do motorista na tela de conferência.
- **FR-006**: O sistema MUST anexar a imagem original da fotografia ao registro do BDO, preservada como prova documental.
- **FR-007**: O sistema MUST sinalizar ao motorista uma possível duplicidade quando o BDO fotografado já corresponder a um registro existente.

### Key Entities

- **BDO**: Documento de origem da entrega. Atributos: campos extraídos do padrão WorldTracer (identificação do passageiro, endereço de entrega, dados do voo/companhia aérea), imagem original anexada, status de conferência (pendente/confirmado).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A extração automática acerta, no mínimo, 90% dos campos obrigatórios do BDO em uma amostra de documentos reais.
- **SC-002**: Nenhum BDO é registrado no sistema sem passar pela tela de conferência.
- **SC-003**: O motorista consegue concluir a captura e a conferência de um BDO em menos tempo do que levaria para digitar manualmente os mesmos dados.

## Assumptions

- O dispositivo do motorista possui câmera com qualidade suficiente para permitir a extração automática dos campos.
- O layout do BDO fotografado segue o padrão de telex do WorldTracer descrito nas evidências do projeto.
- A verificação de duplicidade (Edge Case) é feita por correspondência de campos-chave do BDO (ex. número do documento), não por comparação de imagem.
