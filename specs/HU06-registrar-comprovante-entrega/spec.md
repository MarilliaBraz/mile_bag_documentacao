# Feature Specification: Registrar o comprovante de entrega

**Feature Branch**: `HU06-registrar-comprovante-entrega`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU06 — `../../HUs/motorista/HU06-registrar-comprovante-entrega.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF06), `../../requisitos/03-regras-negocio.md` (RN04), `../../requisitos/04-regras-dominio.md` (RD05, RD06). Ver também a User Story 1 de [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar o comprovante de entrega ao concluir a devolução (Priority: P1)

Ao chegar ao endereço do passageiro e entregar a bagagem, o motorista registra o comprovante de entrega diretamente no aplicativo, encerrando formalmente a entrega.

**Why this priority**: É o evento que fecha o ciclo da entrega e produz a prova exigida pela companhia aérea — sem ele, a entrega nunca é considerada oficialmente concluída (RN04).

**Independent Test**: Pode ser testado com uma entrega em andamento (viagem já iniciada, HU04), registrando o comprovante e verificando que a entrega muda de estado, sem depender de bagagem carona ou de ocorrência.

**Acceptance Scenarios**:

1. **Given** que uma entrega está em andamento e o motorista está no endereço do passageiro, **When** ele registra foto do documento assinado, identificação do recebedor, data, hora e geolocalização, **Then** a entrega passa para o estado "encerrada com sucesso".
2. **Given** que o motorista tenta encerrar a entrega sem preencher um dos campos obrigatórios do comprovante, **When** ele tenta confirmar o registro, **Then** o sistema recusa o registro até que todos os campos estejam completos.
3. **Given** que o motorista está sem conexão com a internet no momento da entrega, **When** ele registra o comprovante, **Then** o registro é aceito e armazenado localmente, sendo sincronizado automaticamente depois (ver HU08).
4. **Given** que uma entrega já foi encerrada com comprovante, **When** o motorista tenta registrar um novo comprovante para a mesma entrega, **Then** o sistema recusa o novo registro.

### Edge Cases

- O que acontece se a captura de geolocalização falhar no momento do registro (ex.: GPS indisponível)? O sistema deve reter o restante do comprovante e sinalizar a ausência da geolocalização, sem impedir uma nova tentativa de captura.
- O que acontece se o recebedor se recusar a assinar o documento? O fluxo de comprovante de entrega não se aplica a esse caso — a situação deve ser tratada como ocorrência de não entrega (HU07), não como comprovante incompleto.
- O que acontece se o motorista registrar o comprovante de uma entrega que já foi reaberta por ocorrência? O novo registro deve ser aceito normalmente, encerrando a nova tentativa.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST exigir, para registrar um comprovante de entrega, foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas.
- **FR-002**: O sistema MUST recusar o registro do comprovante enquanto qualquer um dos campos obrigatórios estiver ausente.
- **FR-003**: O sistema MUST mudar o estado da entrega para "encerrada com sucesso" somente após o registro completo do comprovante.
- **FR-004**: O sistema MUST impedir o registro de mais de um comprovante de entrega para a mesma tentativa de entrega em aberto.
- **FR-005**: O sistema MUST permitir o registro do comprovante mesmo sem conexão com a internet, preservando os dados localmente até a sincronização (ver HU08).

### Key Entities

- **Comprovante de entrega (POD)**: Registro que encerra uma entrega com sucesso — foto do documento assinado, identificação do recebedor, data, hora e geolocalização.
- **Entrega**: Muda de estado para "encerrada com sucesso" ao receber um comprovante de entrega válido.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das entregas concluídas em operação real são encerradas com comprovante completo (foto, recebedor, data, hora e geolocalização) — critério de sucesso do projeto (documento de evidência, seção 3.13).
- **SC-002**: Nenhuma entrega muda para "encerrada com sucesso" sem um comprovante de entrega completo associado.
- **SC-003**: O motorista consegue registrar o comprovante de entrega mesmo em local sem conectividade, sem perda de dados.

## Assumptions

- O documento assinado é fotografado pela própria câmera do dispositivo do motorista, sem necessidade de assinatura digital ou biométrica.
- "Identificação do recebedor" é um campo de texto livre (nome informado), sem validação contra um cadastro de identidade.
