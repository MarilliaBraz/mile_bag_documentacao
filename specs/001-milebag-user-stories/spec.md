# Feature Specification: Milebag — Histórias de Usuário da Operação de Última Milha

**Feature Branch**: `001-milebag-user-stories`

**Created**: 2026-08-20

**Status**: Draft

**Input**: User description: "crie uma spec.md para todas as HUs"

**Fontes**: `../../HUs/` (HU01–HU16) e `../../requisitos/` (RF, RNF, RN, RD, RI), derivados de `../../evidencias/Projeto_de_software_-_ADS_com_TAP.md`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Provisionar um novo tenant e operar o ciclo completo de entrega (Priority: P1)

Uma nova empresa de entrega de bagagens é cadastrada como assinante do sistema. Seu administrador é convidado e passa a poder configurar a operação. Um motorista dessa empresa fotografa um BDO, atribui a entrega a si mesmo, calcula e escolhe a rota até o endereço do passageiro, inicia a viagem (com a rota congelada como base do pagamento) e registra o comprovante de entrega ao concluir. Todo esse fluxo deve continuar funcionando mesmo quando o motorista perde a conexão de internet durante a rota, sincronizando automaticamente ao final.

**Why this priority**: É o núcleo mínimo que torna o sistema operável de ponta a ponta para uma empresa real: sem provisionamento não há tenant nem motorista; sem o ciclo BDO → rota → entrega → comprovante não existe substituição do processo manual em papel e planilha, que é o problema central do projeto (documento de evidência, seções 1.2 e 2.1). Corresponde ao marco de MVP do cronograma do TAP (seção 3.7).

**Independent Test**: Pode ser testado de ponta a ponta cadastrando um tenant, convidando seu administrador, e executando uma entrega completa (captura → atribuição → rota → viagem → comprovante), inclusive em cenário sem conectividade, sem depender de tarifas, pedágios ou fechamento financeiro configurados.

**Acceptance Scenarios**:

1. **Given** que não existe registro do tenant, **When** o administrador da plataforma cadastra uma nova empresa assinante com um plano associado, **Then** o registro do tenant é criado sem exigir qualquer etapa de implantação técnica dedicada.
2. **Given** que o tenant acabou de ser cadastrado, **When** o cadastro é concluído, **Then** um convite é disparado automaticamente ao primeiro administrador indicado, permitindo que ele defina suas credenciais de acesso.
3. **Given** que o motorista está autenticado no aplicativo, **When** ele fotografa um BDO, **Then** o sistema extrai automaticamente os campos do documento e exibe uma tela de conferência com os valores sugeridos, exigindo confirmação antes de salvar o registro.
4. **Given** que um BDO foi confirmado na tela de conferência, **When** o motorista opta por atribuir a entrega a si mesmo, **Then** a entrega passa a constar na lista de entregas em andamento desse motorista.
5. **Given** que uma entrega foi autoatribuída, **When** o motorista solicita o cálculo da rota, **Then** o sistema apresenta ao menos uma rota com distância, tempo estimado e trechos com pedágio, permitindo escolher entre alternativas sugeridas ou traçar uma rota própria.
6. **Given** que uma rota foi escolhida, **When** o motorista inicia a viagem, **Then** o sistema registra um snapshot imutável com quilometragem de ida e volta, pedágios e tempo estimado, que passa a ser a base exclusiva do cálculo de pagamento.
7. **Given** que a viagem foi concluída no endereço do passageiro, **When** o motorista registra o comprovante de entrega (foto do documento assinado, identificação do recebedor, data, hora e geolocalização), **Then** a entrega muda para o estado de encerrada com sucesso.
8. **Given** que o motorista está sem conexão com a internet durante a rota, **When** ele executa qualquer uma das ações acima (captura, atribuição, rota, viagem, comprovante), **Then** os dados são armazenados localmente no dispositivo e sincronizados automaticamente assim que a conectividade for restabelecida, sem exigir ação manual.

---

### User Story 2 - Fechar financeiramente o período de pagamento e tratar exceções da entrega (Priority: P2)

A empresa configura suas tarifas e as praças de pedágio usadas na região de atuação. Quando uma bagagem adicional (carona) pode ser entregue na mesma viagem, o motorista a vincula à entrega principal. Quando uma entrega não se concretiza, o motorista registra uma ocorrência padronizada, o que reabre o BDO para nova tentativa. Ao final de cada quinzena, a empresa gera o fechamento financeiro com os valores devidos a cada motorista e o reembolso de pedágios, com base nos dados já registrados durante o período. A comercialização do serviço segue planos de assinatura diferenciados por porte de operação.

**Why this priority**: Sem esta camada, o sistema resolve a captura e a execução da entrega, mas não resolve o segundo maior problema relatado no documento de evidência: a apuração manual do pagamento e a divergência recorrente entre o valor calculado pela empresa e o esperado pelo motorista (seção 1.2). Corresponde ao marco "Módulo financeiro" do cronograma do TAP (seção 3.7).

**Independent Test**: Pode ser testado configurando tarifas e pedágios para um tenant já operacional (User Story 1), executando algumas entregas — incluindo ao menos uma com bagagem carona e uma com ocorrência registrada — e gerando o fechamento do período, verificando que os valores batem com o esperado sem ajuste manual.

**Acceptance Scenarios**:

1. **Given** que o administrador do tenant está configurando a operação, **When** ele cadastra uma tabela de tarifa com valores fixos e por quilômetro, **Then** essa tabela pode ser aplicada de forma específica por motorista, por região ou como padrão do tenant.
2. **Given** que o administrador do tenant está configurando a operação, **When** ele cadastra uma praça de pedágio com valor e data de vigência, **Then** o valor cadastrado passa a ser usado no cálculo do reembolso de ida e volta para viagens futuras, sem alterar viagens já congeladas anteriormente.
3. **Given** que uma entrega principal está em andamento, **When** o motorista vincula uma bagagem adicional a ela, **Then** apenas a quilometragem complementar dessa bagagem é considerada para fins de remuneração, não uma viagem completa nova.
4. **Given** que o motorista não conseguiu concluir uma entrega, **When** ele registra uma ocorrência a partir do catálogo padronizado do tenant, **Then** o BDO correspondente é reaberto automaticamente e a entrega não permanece marcada como concluída.
5. **Given** que o período quinzenal se encerrou, **When** o administrador do tenant gera o fechamento, **Then** o sistema emite, separadamente, o relatório de pagamento de entregas e o de reembolso de pedágios, calculados a partir dos snapshots de rota e das tarifas vigentes no período, sem exigir ajuste manual.
6. **Given** que o administrador da plataforma está comercializando o serviço, **When** ele cadastra ou altera o plano de assinatura de um tenant (limite de motoristas ativos e de volume mensal de BDOs), **Then** a alteração é aplicada sem necessidade de reimplantação do sistema.

---

### User Story 3 - Personalizar a operação e a apresentação dos relatórios por tenant (Priority: P3)

Cada empresa assinante pode ajustar, de forma independente das demais, os detalhes operacionais do seu formulário de baixa (motivos de ocorrência, campos obrigatórios, bases aeroportuárias, companhias aéreas atendidas, periodicidade do fechamento) e a identidade visual aplicada aos relatórios que envia às companhias aéreas contratantes.

**Why this priority**: Refina a experiência e a autonomia de configuração de cada tenant, um diferencial citado na análise de mercado do documento de evidência (seção 2.3), mas não é indispensável para que a operação piloto funcione com valores padrão razoáveis definidos nas Users Stories 1 e 2.

**Independent Test**: Pode ser testado alterando as configurações de um tenant já operacional e verificando que apenas esse tenant é afetado, e que os relatórios gerados na User Story 2 refletem a identidade visual configurada.

**Acceptance Scenarios**:

1. **Given** que o administrador do tenant acessa as configurações da empresa, **When** ele altera o catálogo de motivos de ocorrência, os campos obrigatórios do formulário de baixa, as bases aeroportuárias, as companhias aéreas atendidas ou a periodicidade do fechamento, **Then** apenas o seu tenant é afetado por essas alterações.
2. **Given** que o administrador do tenant configurou uma identidade visual (logotipo/cabeçalho), **When** um relatório de fechamento é gerado para esse tenant, **Then** o relatório é emitido com essa identidade visual aplicada automaticamente.

---

### Edge Cases

- O que acontece quando o reconhecimento automático (OCR) não consegue extrair um ou mais campos obrigatórios do BDO? O motorista deve poder preenchê-los manualmente na tela de conferência antes de confirmar.
- O que acontece quando o motorista permanece sem conexão por um período prolongado, acumulando várias entregas não sincronizadas? Todos os registros pendentes devem ser preservados localmente e sincronizados, na ordem em que ocorreram, assim que a conexão retornar.
- O que acontece quando uma bagagem carona é vinculada a uma entrega principal que já foi encerrada com comprovante? A vinculação não deve ser permitida; a bagagem deve ser tratada como uma nova entrega.
- O que acontece quando não existe, no cadastro do tenant, uma praça de pedágio para um trecho identificado na rota? A viagem deve poder ser congelada mesmo assim, sinalizando o trecho sem valor de pedágio cadastrado para tratamento posterior pelo administrador.
- O que acontece quando o tenant atinge o limite de motoristas ativos ou de volume mensal de BDOs do seu plano de assinatura? O sistema deve impedir novas ações que excedam o limite e sinalizar a necessidade de mudança de plano ao administrador do tenant.
- O que acontece quando dois tenants diferentes tentam usar as mesmas bases aeroportuárias ou companhias aéreas atendidas? Cada tenant mantém seu próprio cadastro, sem interferência entre empresas.
- O que acontece quando uma entrega é reaberta por ocorrência mais de uma vez? O histórico de tentativas anteriores deve ser preservado e visível, e não sobrescrito pela nova tentativa.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir o cadastro de uma empresa assinante (tenant) associada a um plano de assinatura, sem exigir etapa de implantação técnica dedicada.
- **FR-002**: O sistema MUST enviar, automaticamente ao final do cadastro de um tenant, um convite ao primeiro administrador indicado, permitindo a ele definir suas credenciais de acesso.
- **FR-003**: O sistema MUST permitir o registro do BDO por fotografia, extraindo automaticamente os campos do documento e exigindo confirmação do operador em tela de conferência antes de salvar o registro.
- **FR-004**: O sistema MUST preservar a imagem original do BDO anexada ao registro correspondente, como prova documental.
- **FR-005**: Motoristas MUST ser capazes de atribuir a si mesmos uma entrega no momento em que o BDO é confirmado.
- **FR-006**: O sistema MUST calcular ao menos uma rota sugerida (distância, tempo estimado e trechos com pedágio) para cada entrega e permitir a escolha entre rotas alternativas ou o traçado de uma rota própria.
- **FR-007**: O sistema MUST registrar, no início de cada viagem, um snapshot imutável com quilometragem de ida e volta, pedágios e tempo estimado, que passa a ser a única base para o cálculo do pagamento dessa viagem.
- **FR-008**: O sistema MUST permitir vincular uma ou mais bagagens adicionais (carona) a uma entrega principal em andamento, remunerando apenas a quilometragem complementar decorrente delas.
- **FR-009**: O sistema MUST impedir a vinculação de bagagem carona a uma entrega principal já encerrada com comprovante.
- **FR-010**: O sistema MUST registrar o comprovante de entrega (foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas) como condição para que uma entrega seja considerada encerrada com sucesso.
- **FR-011**: O sistema MUST permitir o registro de uma ocorrência padronizada quando a entrega não se concretizar, reabrindo automaticamente o BDO correspondente e preservando o histórico de tentativas anteriores.
- **FR-012**: O sistema MUST permitir que cada tenant configure suas próprias tabelas de tarifa, com valores fixos e por quilômetro, aplicáveis por motorista, por região ou como padrão do tenant.
- **FR-013**: O sistema MUST permitir que cada tenant cadastre praças de pedágio com valor e data de vigência, preservando valores anteriores para não afetar viagens já congeladas.
- **FR-014**: O sistema MUST calcular o reembolso de pedágio considerando sempre o trajeto de ida e de volta.
- **FR-015**: O sistema MUST gerar, para cada período configurado (por padrão, quinzenal), um relatório de pagamento de entregas e um relatório de reembolso de pedágios, calculados separadamente e sem exigir ajuste manual.
- **FR-016**: O sistema MUST permitir que o administrador da plataforma cadastre e altere planos de assinatura, definindo limites de motoristas ativos e de volume mensal de BDOs por tenant, sem exigir reimplantação.
- **FR-017**: O sistema MUST impedir que um tenant exceda os limites de motoristas ativos ou de volume mensal de BDOs definidos pelo seu plano vigente, sinalizando a necessidade de mudança de plano.
- **FR-018**: O sistema MUST permitir que cada tenant configure, de forma independente dos demais, seus motivos de ocorrência, campos obrigatórios do formulário de baixa, bases aeroportuárias, companhias aéreas atendidas e periodicidade do fechamento.
- **FR-019**: O sistema MUST permitir que cada tenant configure uma identidade visual própria (logotipo/cabeçalho), aplicada automaticamente aos relatórios de fechamento gerados para esse tenant.
- **FR-020**: A aplicação do motorista MUST permitir capturar BDOs, calcular/escolher rotas, iniciar viagens, registrar comprovantes e registrar ocorrências mesmo sem conexão com a internet, armazenando os dados localmente.
- **FR-021**: O sistema MUST sincronizar automaticamente, na ordem em que ocorreram, todos os dados capturados offline assim que a conectividade do dispositivo for restabelecida, sem exigir ação manual do motorista.
- **FR-022**: O sistema MUST garantir que os dados de um tenant nunca sejam visíveis ou acessíveis a outro tenant, em nenhuma das funcionalidades acima.

### Key Entities

- **Tenant**: Empresa terceirizada de entrega domiciliar de bagagens, assinante do sistema. Possui motoristas, bases aeroportuárias, contratos, tabelas de tarifa e configurações próprias, isolados dos demais tenants.
- **Plano de assinatura**: Delimita, por tenant, o número de motoristas ativos e o volume mensal de BDOs processáveis. Cada tenant está associado a exatamente um plano vigente.
- **BDO (Baggage Delivery Order)**: Documento de origem de uma entrega, emitido pela companhia aérea, capturado por fotografia com extração automática de campos e imagem original preservada.
- **Entrega**: Unidade de trabalho que liga um BDO a um motorista, uma rota, um comprovante (ou uma ocorrência) e, eventualmente, bagagens carona vinculadas.
- **Bagagem carona**: Bagagem adicional vinculada a uma entrega principal, remunerada apenas pela quilometragem complementar.
- **Viagem / snapshot de rota**: Registro imutável, criado no início da viagem, com quilometragem de ida e volta, pedágios e tempo estimado — base exclusiva do cálculo de pagamento.
- **Comprovante de entrega (POD)**: Registro que encerra uma entrega com sucesso, composto por foto do documento assinado, identificação do recebedor, data, hora e geolocalização.
- **Ocorrência**: Evento padronizado, configurável por tenant, registrado quando uma entrega não se concretiza; reabre o BDO correspondente.
- **Tabela de tarifa**: Conjunto de valores fixos e por quilômetro, configurável por tenant, motorista ou região, usado no cálculo do pagamento.
- **Praça de pedágio**: Cadastro por tenant de um ponto de cobrança de pedágio, com valor e data de vigência.
- **Fechamento**: Apuração periódica (por padrão quinzenal) que agrega os valores de pagamento de entregas e de reembolso de pedágios de um tenant, calculados separadamente.
- **Usuário / Perfil**: Pessoa com acesso ao sistema, associada a um tenant e a um perfil (administrador do tenant, motorista, ou administrador da plataforma) que determina suas permissões.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um novo tenant consegue ser cadastrado e seu primeiro administrador convidado e ativo em até algumas horas, sem qualquer etapa de implantação técnica dedicada.
- **SC-002**: A extração automática dos campos obrigatórios do BDO é correta em, no mínimo, 90% dos casos, em amostra de documentos reais.
- **SC-003**: 100% das entregas concluídas em operação real são encerradas com comprovante completo (foto, recebedor, data, hora e geolocalização) registrado.
- **SC-004**: O relatório de fechamento periódico é gerado sem qualquer ajuste manual e seus valores coincidem com a apuração equivalente feita fora do sistema (planilha).
- **SC-005**: Nenhum caso de acesso a dados de um tenant por outro tenant é observado em testes de isolamento realizados a cada novo incremento do sistema.
- **SC-006**: O tempo necessário para registrar uma ordem de entrega (do BDO em papel até o registro no sistema) é reduzido em relação ao processo manual anterior (digitação em planilha).
- **SC-007**: Ao menos 80% dos usuários (motoristas e administradores de tenant) relatam satisfação com o sistema em teste de aceitação.
- **SC-008**: Motoristas conseguem concluir toda a sequência de captura do BDO até o registro do comprovante mesmo em trechos sem conectividade, sem perda de dados, com sincronização automática ao retomar a conexão.

## Assumptions

- Cada tenant opera com um único plano de assinatura vigente por vez; a coexistência de múltiplos planos simultâneos para o mesmo tenant está fora do escopo desta especificação.
- O período padrão de fechamento é quinzenal, podendo ser ajustado por tenant conforme a User Story 3; não há suporte a fechamentos ad-hoc fora do período configurado.
- Cada BDO corresponde a uma bagagem; bagagens adicionais entregues na mesma viagem são tratadas como bagagem carona vinculada a uma entrega principal, não como múltiplos BDOs independentes na mesma viagem.
- O motorista possui um dispositivo com câmera e GPS, usados para a captura do BDO, do comprovante e da geolocalização.
- A conectividade móvel está disponível na maior parte das rotas, sendo as interrupções tratadas pela operação offline com sincronização automática (User Story 1).
- O primeiro administrador de um novo tenant é responsável por completar a configuração inicial descrita na User Story 3 antes ou durante o início da operação, usando valores padrão razoáveis até lá.
