# MILEBAG - Gestão de Última Milha de Bagagens

**Faculdade SENAI FATESG**
Curso Superior de Tecnologia em Análise e Desenvolvimento de Sistemas

**Autora:** Marillia Braz Neves Brito
Goiânia, 2026

---

## 1 Introdução

O extravio temporário de bagagens é uma ocorrência rotineira no transporte aéreo. Quando a mala não embarca no mesmo voo do passageiro, a companhia aérea registra a irregularidade no WorldTracer, sistema padrão da IATA (International Air Transport Association), e assume a obrigação de devolvê-la no endereço indicado. Localizada e liberada a bagagem, a devolução domiciliar é quase sempre executada por empresas terceirizadas de entrega, que mantêm motoristas em base nos aeroportos e operam sob contrato com uma ou mais companhias.

### 1.1 Tema e delimitação do estudo

Este trabalho tem como tema o desenvolvimento de um sistema web, no modelo SaaS (Software as a Service) multi-tenant, para a gestão da última milha da devolução de bagagens extraviadas no transporte aéreo brasileiro.

O estudo delimita-se ao trecho operacional que se inicia quando a companhia aérea localiza e libera a bagagem irregular e se encerra na entrega comprovada ao passageiro, executado por empresas terceirizadas de entrega domiciliar. Ficam, portanto, fora do recorte as etapas anteriores da cadeia (check-in, manuseio em solo, rastreio da bagagem perdida e o próprio processo de localização conduzido pela companhia aérea), bem como o tratamento de indenizações e a relação contratual entre passageiro e transportador aéreo.

Do ponto de vista da área de conhecimento, o estudo situa-se no campo da Engenharia de Software aplicada à logística de última milha, compreendendo o levantamento de requisitos, a modelagem, o projeto arquitetural e o desenvolvimento de uma aplicação web. A escolha do modelo SaaS multi-tenant é, em si, uma decisão arquitetural do projeto: uma única instância da aplicação atende simultaneamente a várias empresas de entrega (os tenants), com isolamento lógico dos dados, parametrização própria e ciclo de vida independente para cada uma delas.

### 1.2 Problema

No cenário brasileiro, a ordem de entrega chega à empresa terceirizada em papel: um telex do WorldTracer impresso no balcão da companhia aérea, conhecido no setor como BDO (Baggage Delivery Order). A partir daí, a operação é conduzida com planilhas eletrônicas, mensagens instantâneas e anotações manuais. Os dados do passageiro e o endereço são redigitados; a quilometragem é consultada avulsamente em aplicativos de mapa; o comprovante de entrega circula como fotografia solta em grupos de mensagem; e a apuração do que se deve pagar a cada motorista é remontada manualmente ao final de cada período.

Três consequências decorrem desse arranjo. A primeira é o retrabalho e o erro na entrada de dados, que se propagam até a prestação de contas à companhia aérea. A segunda é a ausência de rastreabilidade do cálculo: como a quilometragem usada no pagamento não fica registrada junto à viagem, a divergência entre o valor apurado pela empresa e o esperado pelo motorista é recorrente e insolúvel. A terceira é a fragilidade da prova de entrega, exigida pela contratante em caso de contestação e cada vez mais relevante diante da Resolução 753 da IATA, que determina o rastreamento da bagagem em todas as etapas, inclusive na devolução ao passageiro.

Há ainda um problema de acesso à tecnologia. As empresas que executam esse serviço no Brasil são, em sua maioria, de pequeno e médio porte, com frota reduzida e sem estrutura de tecnologia da informação própria. Soluções instaladas individualmente, com licenciamento por implantação, são economicamente inviáveis nessa escala.

**Pergunta que orienta a investigação:** como projetar uma aplicação web em modelo SaaS multi-tenant capaz de digitalizar o trecho de última milha da devolução de bagagens, da liberação pela companhia aérea à entrega comprovada, atendendo simultaneamente a múltiplas empresas de entrega, com isolamento de dados, parametrização independente e custo de adoção compatível com o porte desses operadores?

### 1.3 Objetivos

#### 1.3.1 Objetivo geral

Desenvolver uma aplicação web, em arquitetura SaaS multi-tenant, para a gestão da última milha da devolução de bagagens extraviadas, cobrindo o ciclo que vai da liberação da bagagem pela companhia aérea até a comprovação documental da entrega ao passageiro e o consequente fechamento financeiro da operação.

#### 1.3.2 Objetivos específicos

- Levantar os requisitos funcionais e não funcionais do processo de devolução domiciliar junto a empresas de entrega e motoristas atuantes no setor;
- Analisar as soluções já disponíveis no mercado, em especial o AtivMob, utilizado por empresas que prestam esse serviço, identificando as lacunas funcionais que justificam a proposta;
- Definir a estratégia de multi-tenancy, o modelo de isolamento de dados entre tenants e o processo de onboarding de novas empresas assinantes;
- Modelar o domínio da devolução de bagagens em diagramas de classe e no modelo de dados relacional, incorporando o discriminador de tenant;
- Projetar e implementar a extração automática dos dados do BDO por reconhecimento óptico de caracteres, com conferência assistida;
- Implementar o cálculo de distância, rota e pedágio por meio de serviço de mapas, com congelamento dos valores utilizados no pagamento;
- Implementar o registro do comprovante de entrega com fotografia, data, hora e geolocalização;
- Implementar o motor de tarifas parametrizável por tenant, por motorista e por região, e a emissão automática dos relatórios periódicos de pagamento e de reembolso de pedágios;
- Validar a solução por meio de teste piloto em operação real e avaliar os resultados frente aos critérios de sucesso definidos.

### 1.4 Justificativa

A escolha do tema se justifica pela distância entre as ferramentas hoje disponíveis e a forma como a operação brasileira efetivamente acontece.

No mercado nacional já existe o AtivMob, plataforma de registro e gestão de atividades em campo (entregas, coletas, visitas e inspeções) efetivamente utilizada por empresas que prestam o serviço de devolução de bagagens. Trata-se de solução madura na camada de campo: caixa de atividades do motorista, rastreamento em tempo real, foto de comprovante na baixa, registro de ocorrências, formulários configuráveis e vínculo do usuário por convite do administrador. No plano internacional, a SITA oferece o WorldTracer Bag Delivery Service, que recebe eletronicamente a ordem de entrega da companhia aérea e mantém aplicativo do motorista com assinatura de recebimento.

Nenhuma das duas soluções, contudo, cobre o trecho aqui delimitado de ponta a ponta. O WorldTracer BDS pressupõe integração eletrônica com a companhia aérea, condição que raramente se verifica na relação com o pequeno operador brasileiro, que continua recebendo o BDO impresso. O AtivMob, por ser uma plataforma horizontal de atividades, não interpreta o telex do WorldTracer, não conhece a figura da bagagem carona, não aplica tabelas de tarifa negociadas caso a caso e não produz o fechamento financeiro do período. O resultado prático é conhecido: a empresa digitaliza a etapa de campo e mantém, em paralelo, as mesmas planilhas de sempre para a etapa documental e financeira.

A originalidade da proposta está no recorte e no modelo de entrega. No recorte, porque o Milebag não pretende ser mais um gestor genérico de atividades, e sim um sistema verticalizado no domínio da bagagem irregular, aderente ao documento que a companhia aérea de fato entrega ao operador no Brasil. No modelo, porque a arquitetura multi-tenant permite que empresas de pequeno e médio porte (justamente as que não sustentariam uma implantação dedicada) assinem o serviço e comecem a operar em poucas horas, compartilhando a mesma infraestrutura sem compartilhar dados. A atualidade do tema, por fim, é reforçada pela exigência regulatória de rastreamento da devolução e pela consolidação do modelo SaaS como forma predominante de distribuição de software corporativo.

### 1.5 Procedimentos metodológicos

Quanto à natureza, a pesquisa é aplicada; quanto aos objetivos, exploratória e descritiva; quanto ao procedimento, caracteriza-se como pesquisa-desenvolvimento, com estudo de caso em uma empresa de entrega domiciliar de bagagens sediada em Goiânia. O levantamento de requisitos combinou entrevistas semiestruturadas com a administração e com motoristas, observação direta da operação no aeroporto e análise documental de BDOs reais. O desenvolvimento seguiu abordagem iterativa e incremental, com validação de cada incremento em campo antes do início do seguinte.

---

## 2 Escopo do projeto

### 2.1 Situação atual

O processo em estudo começa quando a companhia aérea localiza a bagagem irregular, confirma sua chegada ao aeroporto de destino e a libera para devolução, emitindo o BDO. A partir desse ponto, a empresa terceirizada assume a responsabilidade operacional: retira as malas, organiza as rotas do dia, executa os deslocamentos, entrega ao passageiro e devolve à contratante a comprovação do serviço prestado.

Hoje, esse trecho é conduzido sem sistema próprio. O documento é fotografado ou anotado, os endereços são digitados em planilhas, as rotas são montadas pela experiência do motorista e os comprovantes trafegam por aplicativos de mensagem. Parte das empresas do setor já adota o AtivMob para padronizar a baixa da atividade e o rastreamento em campo, o que resolve a visibilidade da execução, mas mantém intactos os dois gargalos administrativos: a entrada dos dados do BDO continua manual e a apuração do pagamento permanece fora do sistema.

Não existe, nesse cenário, um registro único e auditável que ligue o documento emitido pela companhia aérea, o percurso efetivamente realizado, a prova de entrega e o valor pago ao motorista. É essa cadeia que o projeto se propõe a fechar.

### 2.2 Objetivos do sistema

#### 2.2.1 Digitalizar a entrada da ordem de entrega

Substituir a digitação pela captura fotográfica do BDO com extração automática dos campos do padrão WorldTracer, mantendo tela de conferência obrigatória (o reconhecimento sugere, o operador confirma) e preservando a imagem original anexada ao registro como prova documental.

#### 2.2.2 Tornar o cálculo do pagamento auditável

Registrar, no início da viagem, um snapshot da rota escolhida (quilometragem de ida e volta, praças de pedágio e tempo estimado), de modo que o valor pago seja sempre reconstituível a partir de um dado congelado, e não de uma consulta futura ao serviço de mapas.

#### 2.2.3 Dar autonomia operacional ao motorista

Permitir a autoatribuição do BDO no momento da fotografia, a escolha entre as rotas alternativas sugeridas pelo serviço de mapas ou o traçado de rota própria, e o registro de ocorrências padronizadas quando a entrega não se concretiza.

#### 2.2.4 Comprovar a entrega ao passageiro

Encerrar o ciclo apenas mediante registro do comprovante de entrega com fotografia do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas, disponibilizando esse conjunto para prestação de contas à companhia aérea.

#### 2.2.5 Operar como serviço multiempresa

Atender, em uma única instância da aplicação, a diversas empresas de entrega, cada uma com seus motoristas, bases aeroportuárias, contratos e tabelas de tarifa, sem que os dados de uma sejam acessíveis a outra e sem necessidade de implantação dedicada.

### 2.3 Análise das soluções existentes no mercado

O levantamento identificou três categorias de solução concorrentes ou parcialmente concorrentes: a plataforma vertical da própria indústria aérea (WorldTracer BDS), as plataformas horizontais de gestão de atividades em campo, categoria em que se destaca o AtivMob, por já ser utilizado pelas empresas do setor, e as plataformas genéricas de last mile, voltadas ao comércio eletrônico.

**Quadro 1 — Comparativo entre as soluções de mercado e o Milebag**

| Característica | AtivMob | WorldTracer BDS (SITA) | Milebag (proposto) |
|---|---|---|---|
| Origem da ordem de serviço | Cadastro manual ou integração pontual | Integração eletrônica com a cia. aérea | Foto do BDO impresso com OCR |
| Leitura do telex WorldTracer | Não possui | Não se aplica (dado nativo) | Sim, com conferência assistida |
| Verticalização no domínio bagagem | Não (plataforma horizontal) | Sim | Sim |
| Rastreamento em tempo real | Sim | Sim | Sim |
| Comprovante de entrega (POD) | Sim | Sim | Sim, com foto e geolocalização |
| Registro de ocorrências | Sim, configurável | Sim | Sim, com reabertura do BDO |
| Modo offline | Sim | Parcial | Sim, com sincronização automática |
| Tarifa por motorista e região | Não | Não | Sim, parametrizável por tenant |
| Bagagem carona | Não | Não | Sim, com km complementar |
| Fechamento financeiro do período | Não | Não | Sim, entregas e pedágios separados |
| Modelo de entrega | Assinatura | Contrato com associados IATA | SaaS multi-tenant por assinatura |
| Autonomia de configuração do cliente | Parcial | Baixa | Alta (parametrização por tenant) |

Duas conclusões orientam o projeto. A primeira é funcional: o diferencial defensável não está em rastrear motoristas (função já bem resolvida pelo AtivMob), mas em ligar o documento da companhia aérea ao pagamento do motorista dentro de um mesmo sistema. A segunda é arquitetural: como o AtivMob não expõe API pública documentada e, ainda que expusesse, não cobriria as camadas documental e financeira, optou-se por replicar nativamente as boas práticas de campo já consagradas por ele, evitando dependência de terceiro em um processo central da operação.

### 2.4 Modelo SaaS multi-tenant adotado

No Milebag, cada empresa terceirizada de entrega constitui um tenant. Todos os tenants são atendidos por uma única instância da aplicação e por um único banco de dados, no qual as tabelas do domínio carregam um discriminador de propriedade e o isolamento é imposto por políticas de segurança em nível de linha (Row Level Security), ativadas a partir da identidade do tenant contida no token de autenticação.

**Quadro 2 — Estratégias de isolamento em aplicações multi-tenant**

| Estratégia | Descrição | Vantagens | Motivo da não adoção |
|---|---|---|---|
| Banco por tenant | Uma base de dados isolada para cada empresa | Isolamento físico e restauração individual | Custo e esforço de manutenção incompatíveis com o porte dos assinantes |
| Esquema por tenant | Base única com um schema por empresa | Bom isolamento e migração controlada | Degradação da manutenção conforme cresce o número de tenants |
| Tabelas compartilhadas (adotada) | Base e esquema únicos, com coluna discriminadora e Row Level Security | Menor custo por tenant, atualização única e provisionamento imediato | — |

São consequências diretas dessa decisão: o provisionamento de uma nova empresa não exige implantação de infraestrutura, apenas a criação do registro do tenant e o convite ao primeiro administrador; a evolução do sistema é aplicada simultaneamente a todos os assinantes; e o isolamento passa a ser responsabilidade explícita da camada de dados e da camada de autenticação, tornando-se requisito não funcional crítico, verificado por testes automatizados de vazamento entre tenants.

A parametrização por tenant abrange tabelas de tarifa, motivos de ocorrência, campos obrigatórios no formulário de baixa, bases aeroportuárias, companhias aéreas atendidas, periodicidade do fechamento e identidade visual dos relatórios. A comercialização prevê planos de assinatura diferenciados por número de motoristas ativos e volume mensal de BDOs processados.

### 2.5 Tecnologias utilizadas

**Quadro 3 — Tecnologias previstas para o desenvolvimento**

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Linguagem do back-end | Java 21 (LTS) | Domínio prévio da desenvolvedora; tipagem forte, maturidade da plataforma e suporte de longo prazo |
| Serviço de aplicação | Spring Boot 3 (Spring MVC) | Arquitetura em camadas, injeção de dependências e filtros e interceptadores que permitem estabelecer o contexto de tenant a cada requisição |
| Persistência | Spring Data JPA com Hibernate | Mapeamento objeto-relacional e suporte a filtros aplicados de forma transversal às entidades do domínio |
| Versionamento do banco | Flyway | Migrações versionadas aplicadas de uma só vez a todos os tenants, coerentes com a base compartilhada |
| Segurança | Spring Security com JWT contendo claim de tenant | Autenticação, autorização por perfil e propagação da identidade do tenant a todas as camadas |
| Build e dependências | Maven | Padronização do ciclo de compilação, testes e empacotamento |
| Aplicação web (back-office) | React com TypeScript | Domínio prévio da desenvolvedora; interface responsiva para o administrador do tenant e para o provedor, com tipagem estática nas regras de exibição de valores |
| Aplicação do motorista | Aplicação web progressiva (PWA) em React | Instalável no dispositivo, com acesso à câmera e ao GPS e armazenamento local para operação offline, sem custo de publicação em lojas |
| Banco de dados | PostgreSQL com PostGIS | Integridade referencial, suporte a dados geográficos e Row Level Security nativo, base do isolamento entre tenants |
| OCR | Tesseract OCR 5 (Apache 2.0) com Tess4J | Motor de código aberto executado no próprio servidor, sem custo por página e treinável para o padrão monoespaçado do telex WorldTracer |
| Rotas e pedágios | Valhalla ou OSRM sobre dados do OpenStreetMap | Rotas alternativas, quilometragem e tempo estimado sem custo por requisição; identificação dos trechos com pedágio, cujos valores vêm de tabela parametrizável por tenant |
| Armazenamento de imagens | Volume em disco com prefixo por tenant, migrável para SeaweedFS | Segregação das imagens de BDO e de comprovantes, com acesso encapsulado em repositório que permite troca posterior por armazenamento de objetos compatível com S3 |
| Relatórios | Apache PDFBox ou JasperReports Library | Emissão dos relatórios periódicos de entregas e de pedágios com identidade visual por tenant |
| Testes | JUnit 5, Mockito e Testcontainers | Testes unitários, de integração e de isolamento entre tenants contra instância real do PostgreSQL |
| Documentação da API | OpenAPI (Springdoc) | Especificação gerada a partir do código, atendendo ao requisito de API documentada |
| Infraestrutura | Contêineres Docker; no piloto, em servidor local da empresa parceira | Padronização de ambiente e implantação reprodutível, sem custo de nuvem no piloto e com caminho de migração para escalonamento horizontal na expansão |

### 2.6 Delimitação do escopo do sistema

Não integram esta versão: integração eletrônica direta com os sistemas das companhias aéreas; emissão de documentos fiscais; processamento de cobrança automática das assinaturas por meio de gateway de pagamento; roteirização automática com otimização de múltiplas paradas; aplicativo destinado ao passageiro final; e tratamento de indenizações por extravio definitivo, matéria alheia ao trecho operacional delimitado.

---

## 3 TAP — Termo de Abertura do Projeto

### 3.1 Identificação do projeto

- **Nome do Projeto:** Milebag – Gestão de Última Milha de Bagagens
- **Código/ID do Projeto:** MLB-2026-01
- **Cliente/Organização:** Empresa terceirizada de entrega domiciliar de bagagens sediada em Goiânia (GO), participante do estudo de caso; no modelo SaaS, as demais empresas de entrega assinantes do serviço.
- **Patrocinador (Sponsor):** Direção da empresa Mile Hub Ltda, Inácio Barros Brito.
- **Gerente do Projeto:** Marillia Braz Neves Brito
- **Equipe do Projeto:**
  - Marillia Braz Neves Brito – Gerente do projeto, analista e desenvolvedora full-stack (back-end, front-end e banco de dados).
  - Professor(a) orientador(a) – Orientação técnica e acadêmica.
  - Administração da empresa parceira – Fornecimento de requisitos e validação das regras de negócio.
  - Motoristas da empresa parceira – Validação em campo e teste de usabilidade.
- **Data de Início:** 10/08/2026
- **Data Prevista de Término:** 07/12/2026
- **Versão do Documento:** 1.0

### 3.2 Contexto e justificativa

O extravio temporário de bagagens é ocorrência rotineira no transporte aéreo. Quando a mala não embarca no mesmo voo do passageiro, a companhia aérea registra a irregularidade no WorldTracer, sistema padrão da IATA, e assume a obrigação de devolvê-la no endereço indicado. Localizada e liberada a bagagem, a devolução domiciliar é executada por empresas terceirizadas, que mantêm motoristas em base nos aeroportos e operam sob contrato com uma ou mais companhias.

No cenário brasileiro, a ordem de entrega chega à empresa terceirizada em papel: um telex do WorldTracer impresso no balcão da companhia aérea, conhecido no setor como BDO (Baggage Delivery Order). A partir daí, a operação é conduzida com planilhas eletrônicas, mensagens instantâneas e anotações manuais. Os dados do passageiro e o endereço são redigitados; a quilometragem é consultada avulsamente em aplicativos de mapa; o comprovante de entrega circula como fotografia solta em grupos de mensagem; e a apuração do que se deve pagar a cada motorista é remontada manualmente ao final de cada período.

Desse arranjo decorrem o retrabalho e o erro na entrada de dados, a ausência de rastreabilidade do cálculo do pagamento e a fragilidade da prova de entrega, exigida pela contratante em caso de contestação e cada vez mais relevante diante da Resolução 753 da IATA. Soma-se a isso o problema de acesso à tecnologia: as empresas que executam esse serviço no Brasil são, em sua maioria, de pequeno e médio porte, sem estrutura de tecnologia da informação própria, para as quais soluções instaladas individualmente são economicamente inviáveis. O projeto Milebag propõe uma aplicação web em modelo SaaS multi-tenant que digitaliza esse trecho de última milha, ligando o documento emitido pela companhia aérea ao pagamento do motorista dentro de um mesmo sistema.

### 3.3 Objetivo do projeto

Desenvolver uma aplicação web em arquitetura SaaS multi-tenant para a gestão da última milha da devolução de bagagens extraviadas, cobrindo o ciclo que se inicia na liberação da bagagem pela companhia aérea e se encerra na comprovação documental da entrega ao passageiro e no consequente fechamento financeiro quinzenal, atendendo simultaneamente a múltiplas empresas de entrega com isolamento de dados e parametrização independente.

O objetivo será considerado alcançado quando o sistema estiver implantado e validado em teste piloto com uma empresa de entrega em operação real, até 7 de dezembro de 2026, conforme os critérios de sucesso definidos na seção 3.13.

### 3.4 Benefícios esperados

- Eliminar a redigitação dos dados do passageiro e do endereço, reduzindo o retrabalho e o erro na entrada de dados.
- Tornar o cálculo do pagamento auditável, por meio do congelamento da quilometragem e dos pedágios no início da viagem.
- Encerrar a divergência recorrente entre o valor apurado pela empresa e o esperado pelo motorista.
- Fortalecer a prova de entrega, com fotografia, identificação do recebedor, data, hora e geolocalização.
- Centralizar em um único registro o documento da companhia aérea, o percurso realizado, o comprovante e o valor pago.
- Reduzir o tempo de apuração do fechamento quinzenal de pagamentos e de reembolso de pedágios.
- Viabilizar o acesso de empresas de pequeno e médio porte à solução, sem implantação dedicada.

### 3.5 Escopo de alto nível

**Incluído no projeto**

- Cadastro de empresas assinantes (tenants), planos, assinaturas e usuários.
- Captura do BDO por fotografia, com extração automática dos campos por OCR e conferência assistida.
- Autoatribuição da entrega pelo motorista no momento da captura do documento.
- Cálculo de rota, quilometragem, tempo e pedágios por serviço de mapas, com escolha de rota alternativa ou própria.
- Congelamento (snapshot) da rota utilizada como base do pagamento.
- Tratamento de bagagem carona, com cobrança apenas da quilometragem complementar.
- Registro do comprovante de entrega (POD) com foto, recebedor, data, hora e geolocalização.
- Registro de ocorrências padronizadas e reabertura do BDO quando a entrega não se concretiza.
- Motor de tarifas parametrizável por tenant, por motorista e por região.
- Cadastro de praças de pedágio e respectivos valores, utilizados no cálculo do reembolso.
- Fechamento quinzenal com relatórios de pagamento de entregas e de reembolso de pedágios.
- Aplicação do motorista em modo offline, com sincronização automática.

**Fora do escopo**

- Integração eletrônica direta com os sistemas das companhias aéreas.
- Emissão de documentos fiscais.
- Cobrança automática das assinaturas por meio de gateway de pagamento.
- Roteirização automática com otimização de múltiplas paradas.
- Aplicativo destinado ao passageiro final.
- Tratamento de indenizações por extravio definitivo.
- Etapas anteriores da cadeia (check-in, manuseio em solo e localização da bagagem pela companhia aérea).

### 3.6 Requisitos de alto nível

| ID | Requisito | Descrição |
|---|---|---|
| RF01 | Capturar imagem do BDO | O sistema deverá permitir o registro do BDO por fotografia, com extração automática dos campos do padrão WorldTracer e tela de conferência obrigatória. |
| RF02 | Autoatribuir a entrega | O motorista deverá poder atribuir a si a entrega no momento da captura do documento. |
| RF03 | Calcular a rota da entrega | O sistema deverá calcular distância, tempo e trechos com pedágio por meio de serviço de rotas de código aberto sobre dados do OpenStreetMap, permitindo a escolha de rota alternativa ou o traçado de rota própria. |
| RF04 | Congelar a quilometragem da viagem | O sistema deverá congelar a quilometragem e os pedágios no início da viagem, mantendo esses valores como base do pagamento. |
| RF05 | Vincular bagagem carona | O sistema deverá permitir vincular bagagens adicionais a uma entrega principal, remunerando apenas a quilometragem complementar. |
| RF06 | Registrar o comprovante de entrega | O sistema deverá registrar foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas. |
| RF07 | Registrar ocorrências de entrega | O sistema deverá permitir registrar ocorrências padronizadas quando a entrega não se concretizar, com reabertura do BDO. |
| RF08 | Configurar tabelas de tarifa | O sistema deverá permitir configurar tarifas por tenant, por motorista e por região, incluindo valores fixos e por quilômetro. |
| RF09 | Gerar o fechamento quinzenal | O sistema deverá emitir relatórios periódicos de pagamento de entregas e de reembolso de pedágios. |
| RF10 | Gerenciar empresas assinantes | O sistema deverá permitir o cadastro de empresas assinantes, planos e usuários, com convite ao primeiro administrador do tenant. |
| RF11 | Cadastrar praças de pedágio | O sistema deverá permitir o cadastro das praças de pedágio e de seus valores por tenant, aplicados sempre em ida e volta. |
| RNF01 | Isolar os dados entre tenants | Os dados de uma empresa não poderão ser acessíveis a outra, com isolamento imposto em nível de linha no banco de dados. |
| RNF02 | Operar sem conexão | A aplicação do motorista deverá operar sem conexão, sincronizando automaticamente quando a rede for restabelecida. |
| RNF03 | Proteger o acesso e os dados pessoais | Autenticação por token, autorização por perfil e tratamento dos dados pessoais do passageiro conforme a LGPD. |
| RNF04 | Utilizar a aplicação em campo | A interface do motorista deverá ser utilizável em smartphone, durante a rota e com o mínimo de toques por entrega. |
| RNF05 | Documentar a interface de programação | A interface de programação deverá ser documentada em padrão OpenAPI. |

### 3.7 Principais entregas e marcos

| Entrega | Descrição | Previsão |
|---|---|---|
| Levantamento e modelagem | Entrevistas com administração e motoristas, análise de BDOs reais, requisitos, diagramas de classe e modelo de dados | 04/09/2026 |
| Protótipo e ambiente | Protótipos de baixa fidelidade e ambiente de desenvolvimento em contêineres com banco, serviço de rotas e OCR locais | 18/09/2026 |
| MVP | Primeira versão funcional: captura do BDO com OCR, cálculo de rota e comprovante de entrega | 30/10/2026 |
| Módulo financeiro | Motor de tarifas parametrizável, cadastro de pedágios e relatórios do fechamento quinzenal | 13/11/2026 |
| Teste piloto | Validação em operação real e correções decorrentes | 27/11/2026 |
| Entrega final | Sistema implantado e documentação do trabalho concluída | 07/12/2026 |

### 3.8 Stakeholders principais

| Stakeholder | Papel | Interesse/Influência |
|---|---|---|
| Direção da empresa de entrega | Patrocinador | Alto |
| Administração / back-office | Usuário principal | Alto |
| Motoristas | Usuários em campo | Alto |
| Companhias aéreas contratantes | Destinatárias da prestação de contas | Alto |
| Passageiro | Beneficiário final | Médio |
| Gerente do projeto | Gestão e desenvolvimento | Alto |
| Faculdade SENAI FATESG | Orientação e avaliação acadêmica | Alto |
| Provedores de serviços externos | Fornecedores de mapas, OCR e nuvem | Médio |

### 3.9 Premissas, restrições e dependências

**Premissas**

- A empresa parceira disponibilizará acesso à operação, a BDOs reais e aos motoristas para entrevistas e testes.
- A empresa parceira disponibilizará um computador ou servidor próprio para hospedar a aplicação durante o teste piloto, dispensando contratação de nuvem.
- Os motoristas possuirão smartphone com câmera e receptor de GPS.
- Os BDOs recebidos seguirão o formato de telex do padrão WorldTracer.
- Haverá conectividade móvel na maior parte das rotas, sendo as interrupções tratadas pelo modo offline.
- Os dados do OpenStreetMap para Goiás e regiões atendidas terão cobertura viária suficiente para o cálculo de distâncias.
- Os valores das praças de pedágio serão informados pela empresa e mantidos por cadastro manual.

**Restrições**

- O projeto deverá ser executado entre 10 de agosto e 7 de dezembro de 2026, dentro do calendário acadêmico.
- Não haverá investimento financeiro inicial: somente softwares de código aberto e recursos gratuitos poderão ser utilizados, ficando vedada a contratação de serviços pagos ou de licenças comerciais.
- A equipe de desenvolvimento é composta por uma única integrante.
- As tecnologias limitam-se ao domínio prévio da desenvolvedora: Java 21, Spring Boot 3, React com TypeScript e PostgreSQL.
- O reconhecimento de caracteres e o cálculo de rotas serão executados em servidor próprio, com desempenho e acurácia inferiores aos de serviços comerciais equivalentes.
- Os dados do OpenStreetMap são licenciados em ODbL, exigindo atribuição visível nos mapas exibidos.
- Não haverá integração eletrônica com os sistemas das companhias aéreas.
- O tratamento dos dados pessoais dos passageiros deverá observar a LGPD.

**Dependências**

- Dados cartográficos do OpenStreetMap (extrato regional) para cálculo de rotas e distâncias.
- Serviço de rotas de código aberto executado localmente (Valhalla ou OSRM).
- Serviço de geocodificação de endereços de código aberto (Nominatim ou Photon).
- Motor de OCR Tesseract instalado no servidor da aplicação.
- Banco de dados PostgreSQL com PostGIS e suporte a Row Level Security.
- Máquina disponibilizada pela empresa parceira para hospedagem durante o piloto.
- Disponibilidade da empresa parceira para a execução do teste piloto.

### 3.10 Riscos iniciais

| Risco | Probabilidade | Impacto | Resposta Inicial |
|---|---|---|---|
| Acurácia do OCR local inferior à de serviços comerciais | Alta | Médio | Pré-tratamento da imagem, treino do Tesseract com amostras do telex e conferência assistida obrigatória |
| Cobertura incompleta do OpenStreetMap em endereços do interior | Média | Médio | Permitir ajuste manual do ponto de entrega e o traçado de rota própria pelo motorista |
| Indisponibilidade da máquina cedida para hospedagem do piloto | Média | Alto | Manter a aplicação em contêineres, com implantação reprodutível em qualquer máquina e backup diário |
| Vazamento de dados entre tenants | Baixa | Alto | Row Level Security e testes automatizados de isolamento em cada incremento |
| Indisponibilidade de conexão durante as rotas | Alta | Médio | Operação offline com sincronização automática |
| Resistência dos motoristas à adoção do aplicativo | Média | Alto | Envolver motoristas desde o levantamento e realizar testes de usabilidade |
| Prazo de dezessete semanas com equipe de um único integrante | Alta | Alto | Desenvolvimento iterativo com priorização do MVP e acompanhamento semanal |
| Desatualização dos valores de pedágio cadastrados manualmente | Média | Baixo | Registrar data de vigência de cada valor e revisá-los a cada fechamento |
| Divergência entre a quilometragem calculada e a percebida pelo motorista | Média | Médio | Snapshot congelado da rota e possibilidade de rota própria justificada |

### 3.11 Recursos e tecnologias

Em razão da restrição de investimento inicial, toda a pilha tecnológica do projeto piloto é composta por software livre e de código aberto, executado em servidor próprio, sem contratação de serviços pagos ou de licenças comerciais.

- **Equipe:** Uma desenvolvedora full-stack, acumulando a gerência do projeto, com orientação acadêmica da Faculdade SENAI FATESG.
- **Front-end:** React (MIT) com TypeScript (Apache 2.0) e Vite (MIT) no back-office; aplicação web progressiva para o motorista, com service worker gerado por Workbox (MIT), armazenamento local em IndexedDB por meio do Dexie.js (Apache 2.0) e exibição de mapas com Leaflet (BSD) sobre camadas do OpenStreetMap.
- **Back-end:** Java 21 LTS na distribuição Eclipse Temurin (GPLv2 com Classpath Exception), Spring Boot 3, Spring Data JPA, Spring Security com JWT, Flyway Community e Maven, todos sob Apache 2.0.
- **Banco de dados:** PostgreSQL (PostgreSQL License) com a extensão PostGIS (GPLv2) e Row Level Security nativo.
- **OCR:** Tesseract OCR 5 (Apache 2.0), integrado ao back-end pelo Tess4J (Apache 2.0), com pré-processamento das imagens do BDO.
- **Rotas e distâncias:** Motor de rotas de código aberto executado localmente – Valhalla (MIT) ou OSRM (BSD) – sobre extrato regional do OpenStreetMap (ODbL), com geocodificação por Nominatim (GPLv2) ou Photon (Apache 2.0). Os trechos com pedágio são identificados pelo próprio motor de rotas e os valores vêm da tabela de praças cadastrada por tenant.
- **Armazenamento de imagens:** Volume em disco do próprio servidor, com prefixo por tenant e acesso encapsulado por interface de repositório, permitindo migração futura para armazenamento de objetos compatível com S3, como o SeaweedFS (Apache 2.0) ou o Garage (AGPLv3).
- **Relatórios:** Geração de PDF no servidor com Apache PDFBox (Apache 2.0) ou JasperReports Library (LGPLv3).
- **Infraestrutura:** Contêineres Docker Engine (Apache 2.0) ou Podman (Apache 2.0), orquestrados por Compose, com proxy reverso Caddy (Apache 2.0) ou Nginx (BSD) e certificado TLS gratuito emitido pelo Let's Encrypt. Hospedagem em máquina da própria empresa durante o piloto.
- **Testes:** JUnit 5 (EPL 2.0), Mockito (MIT) e Testcontainers (MIT), com documentação da API gerada por Springdoc OpenAPI (Apache 2.0).
- **Ambiente de desenvolvimento:** Todo o back-end será desenvolvido no IntelliJ IDEA, distribuído desde a versão 2025.3 em edição única, com as funcionalidades da antiga Community Edition gratuitas para uso comercial e não comercial e código-fonte publicado sob Apache 2.0. O suporte avançado a Spring pertence à assinatura Ultimate e é dispensável no piloto, podendo ser suprido pela licença acadêmica gratuita da JetBrains ou pelo plugin Spring Explyt (Apache 2.0). O front-end será desenvolvido em VS Code ou VSCodium (MIT).
- **Ferramentas:** Git com repositório em plano gratuito, Penpot (MPL 2.0) para prototipação, draw.io (Apache 2.0) para os diagramas e Bruno ou Hoppscotch (MIT) para testes de API.
- **Licenciamento:** Nenhum componente exige pagamento de licença. Os componentes sob licenças recíprocas (PostGIS, Nominatim, JasperReports) são utilizados como serviços ou bibliotecas separadas, sem incorporação de código ao produto, e os dados do OpenStreetMap são creditados nas telas de mapa conforme exige a ODbL.

### 3.12 Cronograma e orçamento — macro

**Duração estimada:** Dezessete semanas, de 10 de agosto a 7 de dezembro de 2026.

**Principais fases**

1. Levantamento de requisitos e modelagem – 4 semanas.
2. Prototipação e preparação do ambiente em contêineres – 2 semanas.
3. Desenvolvimento do MVP – 6 semanas.
4. Motor de tarifas, pedágios e fechamento quinzenal – 2 semanas.
5. Testes e teste piloto em operação real – 2 semanas.
6. Ajustes, implantação e documentação – 1 semana.

**Orçamento estimado:** R$ 0,00. Por se tratar de projeto piloto, não haverá investimento inicial: a solução será construída exclusivamente com software de código aberto e hospedada em máquina já existente na empresa parceira.

**Principais custos**

- Licenças de software: R$ 0,00 – toda a pilha é de código aberto.
- Hospedagem: R$ 0,00 – servidor próprio da empresa parceira, sem contratação de nuvem.
- Reconhecimento de caracteres: R$ 0,00 – Tesseract executado no próprio servidor.
- Rotas, distâncias e geocodificação: R$ 0,00 – motor de rotas local sobre dados do OpenStreetMap.
- Armazenamento de imagens: R$ 0,00 – disco do próprio servidor.
- Domínio e certificado: R$ 0,00 – subdomínio gratuito e certificado Let's Encrypt.
- Energia elétrica e conectividade do servidor: já incorridas pela empresa parceira.

Passam a existir custos apenas na eventual expansão comercial do serviço, fora do escopo deste piloto: servidor em nuvem, domínio próprio, backup externo e, se for o caso, substituição do motor de OCR por serviço comercial de maior acurácia.

### 3.13 Critérios de sucesso

- MVP implantado e disponível até 7 de dezembro de 2026.
- Extração automática correta de, no mínimo, 90% dos campos obrigatórios do BDO em amostra de documentos reais.
- Cem por cento das entregas realizadas no piloto encerradas com comprovante registrado, contendo foto, recebedor, data, hora e geolocalização.
- Relatório quinzenal de pagamento gerado sem ajuste manual e coincidente com a apuração feita em planilha.
- Nenhuma ocorrência de acesso indevido a dados de outro tenant nos testes automatizados de isolamento.
- Redução do tempo de registro de uma ordem de entrega em relação ao processo manual atual.
- Índice de satisfação superior a 80% entre os usuários no teste de aceitação.
- Execução completa do piloto sem qualquer desembolso, mantendo o custo de licenças e de infraestrutura em R$ 0,00.

### 3.14 Governança e autoridade

- **Patrocinador:** Aprovar o projeto, liberar o acesso à operação e aos documentos, autorizar o teste piloto e as mudanças relevantes de escopo.
- **Gerente do Projeto:** Coordenar cronograma, riscos, entregas e a comunicação com os envolvidos.
- **Product Owner / Responsável pelo Produto:** Priorizar funcionalidades e representar as necessidades da operação; papel exercido pela administração da empresa parceira em conjunto com a gerente do projeto.
- **Equipe de Desenvolvimento:** Projetar, desenvolver, testar e documentar a solução.
- **Orientação acadêmica:** Acompanhar o desenvolvimento e avaliar a aderência do trabalho aos requisitos do curso.
- **Aprovação de mudanças de escopo:** Alterações relevantes deverão ser avaliadas pela gerente do projeto e aprovadas pelo patrocinador, com registro em nova versão deste documento.

### 3.15 Aprovação do projeto

A aprovação deste documento representa a autorização formal para o início do projeto, considerando os objetivos, escopo, entregas, premissas, restrições, riscos e demais condições apresentadas neste documento.

- **Patrocinador:** Inácio Barros Brito — Assinatura: ______________________________________
- **Gerente do Projeto:** Marillia Braz Neves Brito — Assinatura: ______________________________________
- **Data:** ____ / ____ / ______

---

## 4 EAP

*(Inserir EAP)*

## 5 Especificação de requisitos

Lista os requisitos funcionais e não funcionais do sistema.

## 6 Diagramas de classe

Diagramas de classes das principais entidades do sistema relacionadas ao domínio do sistema.

## 7 Ambiente de produção

Define os requisitos mínimos para o cliente e servidor para que o sistema possa funcionar normalmente.

## 8 Especificação de interface

Lista alguns requisitos relacionados à interface com o usuário com a listagem de alguns protótipos de baixa fidelidade.

## 9 Descrição do desenvolvimento do sistema proposto

Descreve, através da apresentação de telas, o desenvolvimento do sistema proposto.

## 10 Considerações finais

Insere as considerações finais e proposta de trabalhos futuros.

## Referências

Utilizada para indicar ao leitor as fontes consultadas para a elaboração do trabalho. São referenciados todos os tipos de materiais, como livros, revistas, folhetos, relatórios, documentos da internet, mapas, manuscritos entre outros.

As referências devem estar em ordem alfabética com alinhamento à esquerda e com espaçamento simples.

Devem ser utilizados os padrões estabelecidos pela ABNT.

---

## Apêndice A – Título do apêndice (opcional)

Documento a parte do texto, mas que o complementa e agrega valor ao trabalho produzido. Esse documento deve ser produzido pelo mesmo autor ou autores do trabalho apresentado.

## Anexo A – Título do anexo (opcional)

Documento a parte do texto, mas que o complementa e agrega valor ao trabalho produzido. Esse documento NÃO DEVE ser produzido pelo mesmo autor ou autores do trabalho apresentado. Trata-se de um documento ou parte de um documento produzido por outro autor, que foi anexado ao trabalho com o objetivo de enriquecer o conteúdo apresentado.

NÃO ESQUECER DE REFERENCIAR DEVIDAMENTE O DOCUMENTO ANEXADO.
