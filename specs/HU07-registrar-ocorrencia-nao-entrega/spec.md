# Feature Specification: Registrar ocorrência de não entrega

**Feature Branch**: `HU07-registrar-ocorrencia-nao-entrega`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU07 — `../../HUs/motorista/HU07-registrar-ocorrencia-nao-entrega.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF07), `../../requisitos/03-regras-negocio.md` (RN05), `../../requisitos/04-regras-dominio.md` (RD05, RD07). Ver também a User Story 2 de [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar ocorrência quando a entrega não se concretiza (Priority: P1)

O motorista chega ao endereço do passageiro, mas não consegue concluir a entrega (ausência do recebedor, endereço não localizado, recusa de recebimento, etc.). Ele registra uma ocorrência padronizada, e o sistema reabre o BDO para uma nova tentativa.

**Why this priority**: Sem esta HU, uma entrega malsucedida fica sem tratamento formal no sistema, obrigando o motorista a recorrer novamente a mensagens ou anotações manuais — exatamente o problema que o projeto busca eliminar.

**Independent Test**: Pode ser testado com uma entrega em andamento, registrando uma ocorrência do catálogo do tenant e verificando que o BDO é reaberto, sem depender de comprovante ou bagagem carona.

**Acceptance Scenarios**:

1. **Given** que uma entrega está em andamento e não pôde ser concluída, **When** o motorista seleciona um motivo do catálogo de ocorrências do tenant e confirma o registro, **Then** o BDO correspondente é reaberto automaticamente para nova tentativa.
2. **Given** que uma ocorrência foi registrada, **When** a entrega é consultada depois, **Then** o histórico mostra a ocorrência anterior, sem sobrescrever ou apagar o registro da tentativa malsucedida.
3. **Given** que o motorista tenta registrar uma ocorrência para uma entrega já encerrada com comprovante de entrega, **When** ele confirma o registro, **Then** o sistema recusa a ocorrência.
4. **Given** que o motorista está sem conexão com a internet, **When** ele registra a ocorrência, **Then** o registro é aceito localmente e sincronizado automaticamente depois (ver HU08).

### Edge Cases

- O que acontece se a entrega for reaberta por ocorrência mais de uma vez? Cada nova ocorrência é registrada e preservada; o histórico completo de tentativas fica visível, e não apenas a mais recente.
- O que acontece se o catálogo de motivos de ocorrência do tenant não tiver nenhum motivo cadastrado? O motorista não deve conseguir registrar a ocorrência até que o administrador do tenant cadastre ao menos um motivo (ver HU11).
- O que acontece se dois motoristas diferentes tentarem registrar uma ocorrência para a mesma entrega ao mesmo tempo? Apenas o primeiro registro válido é aceito; o segundo deve ser tratado como uma tentativa após a reabertura, não como uma ocorrência duplicada da mesma tentativa.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir que o motorista registre uma ocorrência para uma entrega em andamento, selecionando um motivo a partir do catálogo padronizado configurado pelo tenant.
- **FR-002**: O sistema MUST reabrir automaticamente o BDO correspondente ao registrar uma ocorrência.
- **FR-003**: O sistema MUST impedir o registro de uma ocorrência para uma entrega já encerrada com comprovante de entrega.
- **FR-004**: O sistema MUST preservar o histórico de todas as ocorrências e tentativas anteriores de uma entrega, sem sobrescrevê-las quando uma nova tentativa é iniciada.
- **FR-005**: O sistema MUST permitir o registro de ocorrência mesmo sem conexão com a internet, preservando os dados localmente até a sincronização (ver HU08).

### Key Entities

- **Ocorrência**: Evento padronizado, escolhido de um catálogo configurável por tenant, registrado quando uma entrega não se concretiza; associada a uma entrega e reabre o BDO correspondente.
- **Motivo de ocorrência**: Item do catálogo configurável por tenant (ex.: "ausência do recebedor", "endereço não localizado"), usado para padronizar o registro.
- **Entrega**: Muda de estado para "reaberta" ao receber uma ocorrência; preserva o histórico de tentativas anteriores.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Nenhuma entrega malsucedida fica sem registro formal — toda tentativa não concluída resulta em uma ocorrência associada.
- **SC-002**: 100% dos BDOs associados a uma ocorrência são reabertos automaticamente, sem intervenção manual do administrador do tenant.
- **SC-003**: O histórico completo de tentativas de uma entrega (incluindo ocorrências anteriores) fica disponível para consulta, mesmo após uma nova tentativa ser concluída com sucesso.

## Assumptions

- O catálogo de motivos de ocorrência é responsabilidade de configuração de cada tenant (coberta pela HU11); esta HU assume que ao menos um motivo já está cadastrado no tenant em operação.
- Uma ocorrência não exige evidência fotográfica; apenas a seleção do motivo e uma descrição textual opcional.
