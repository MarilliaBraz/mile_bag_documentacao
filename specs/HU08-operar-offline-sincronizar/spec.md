# Feature Specification: Operar offline e sincronizar automaticamente

**Feature Branch**: `HU08-operar-offline-sincronizar`

**Created**: 2026-08-20

**Status**: Draft

**Input**: HU08 — `../../HUs/motorista/HU08-operar-offline-sincronizar.md`

**Fontes**: `../../requisitos/01-requisitos-funcionais.md` (RF14), `../../requisitos/02-requisitos-nao-funcionais.md` (RNF02), `../../requisitos/05-regras-interface.md` (RI05). Ver também a User Story 1 de [`../001-milebag-user-stories/spec.md`](../001-milebag-user-stories/spec.md), que já inclui esta capacidade como parte do ciclo completo de entrega.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Continuar trabalhando sem conexão e sincronizar automaticamente (Priority: P1)

O motorista está em uma rota com conectividade instável ou inexistente. Ele continua capturando BDOs, escolhendo rotas, iniciando viagens, registrando comprovantes e ocorrências normalmente. Assim que o dispositivo volta a ter conexão, tudo o que foi feito offline é enviado ao servidor automaticamente, sem que o motorista precise fazer nada.

**Why this priority**: É um requisito transversal a praticamente todas as demais HUs do motorista (HU01–HU07) — sem ela, qualquer interrupção de rede durante a rota interrompe o trabalho do motorista, reproduzindo o problema que o projeto existe para resolver.

**Independent Test**: Pode ser testado colocando o dispositivo do motorista em modo avião, executando o ciclo completo de uma entrega (captura, atribuição, rota, viagem, comprovante) e depois restaurando a conectividade, verificando que todos os registros chegam ao servidor sem intervenção manual.

**Acceptance Scenarios**:

1. **Given** que o dispositivo do motorista está sem conexão com a internet, **When** ele executa qualquer ação de captura (BDO, atribuição, rota, início de viagem, comprovante, ocorrência), **Then** a ação é aceita normalmente e os dados são armazenados localmente no dispositivo.
2. **Given** que existem registros pendentes de sincronização, **When** o motorista consulta a interface, **Then** ele vê uma indicação clara de que está operando offline e quais registros ainda não foram sincronizados.
3. **Given** que o dispositivo estava offline e volta a ter conexão, **When** a conectividade é restabelecida, **Then** todos os registros pendentes são enviados ao servidor automaticamente, na mesma ordem em que foram criados, sem exigir nenhuma ação do motorista.
4. **Given** que um registro já foi sincronizado com sucesso, **When** o dispositivo tenta reenviá-lo (ex.: por uma reconexão interrompida no meio do envio), **Then** o servidor reconhece o reenvio e não duplica o registro.

### Edge Cases

- O que acontece se o motorista permanecer offline por um período muito longo, acumulando dezenas de registros pendentes? Todos devem ser preservados localmente até a sincronização, sem que o armazenamento local do dispositivo os descarte silenciosamente.
- O que acontece se a conexão cair no meio do envio de um lote de sincronização? Os itens já confirmados pelo servidor não devem ser reenviados; os itens não confirmados devem ser reenviados na próxima tentativa, sem duplicação.
- O que acontece se dois eventos offline relacionados à mesma entrega forem criados fora de ordem no dispositivo (ex.: relógio do aparelho incorreto)? A sincronização deve preservar a ordem de criação registrada localmente, não a ordem de chegada ao servidor.
- O que acontece se o dispositivo do motorista for trocado ou os dados locais forem apagados antes da sincronização? Os registros ainda não sincronizados são perdidos — esse risco é aceito como limitação conhecida, já que não há backup fora do dispositivo para dados ainda não enviados.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: A aplicação do motorista MUST permitir executar todas as ações de captura (BDO, autoatribuição, rota, início de viagem, comprovante de entrega, ocorrência) sem conexão com a internet, armazenando os dados localmente no dispositivo.
- **FR-002**: A interface MUST indicar de forma visível quando o motorista está operando sem conexão.
- **FR-003**: A interface MUST indicar quais registros ainda estão pendentes de sincronização.
- **FR-004**: O sistema MUST sincronizar automaticamente todos os registros pendentes assim que a conectividade do dispositivo for restabelecida, sem exigir ação manual do motorista.
- **FR-005**: O sistema MUST preservar, na sincronização, a ordem cronológica de criação dos registros no dispositivo.
- **FR-006**: O sistema MUST reconhecer o reenvio de um registro já sincronizado anteriormente e não criar um registro duplicado.
- **FR-007**: O sistema MUST preservar localmente todos os registros pendentes até que a sincronização de cada um seja confirmada pelo servidor.

### Key Entities

- **Registro pendente de sincronização**: Representação local (no dispositivo do motorista) de uma ação já executada offline, aguardando envio ao servidor; carrega um identificador próprio, o tipo de ação e os dados necessários para reproduzi-la no servidor.
- **Fila de sincronização**: Conjunto ordenado de registros pendentes de um dispositivo, processado em ordem cronológica quando a conectividade retorna.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O motorista consegue concluir toda a sequência de captura do BDO até o registro do comprovante mesmo em trechos sem conectividade, sem perda de dados.
- **SC-002**: 100% dos registros criados offline chegam ao servidor após a reconexão, sem exigir qualquer ação manual do motorista.
- **SC-003**: Nenhum registro é duplicado no servidor em decorrência de reenvio de sincronização.
- **SC-004**: A ordem cronológica dos eventos de uma mesma entrega é preservada após a sincronização, mesmo quando criados em sequência rápida offline.

## Assumptions

- Cada motorista só cria e sincroniza dados das próprias entregas; não há edição concorrente do mesmo registro por dois motoristas, o que evita a necessidade de resolução de conflitos entre dispositivos.
- A detecção de conectividade e o disparo da sincronização são responsabilidade da aplicação do motorista, não exigem ação do back-office.
- Registros ainda não sincronizados que forem perdidos por troca ou reset do dispositivo são um risco aceito nesta versão (ver Edge Cases); não há mecanismo de backup fora do dispositivo antes da sincronização.
