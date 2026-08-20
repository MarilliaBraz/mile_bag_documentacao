# Fase 0 — Pesquisa e Decisões Técnicas

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Este documento resolve os itens marcados `NEEDS CLARIFICATION` no Technical Context de `plan.md` e documenta a racionalidade das decisões técnicas já implícitas nas evidências do projeto (`evidencias/Projeto_de_software_-_ADS_com_TAP.md`), para que fiquem explícitas e rastreáveis antes do desenho de dados e contratos (Fase 1).

## 1. Framework de teste do front-end

**Decision**: Vitest + React Testing Library, com Playwright para os fluxos críticos de ponta a ponta (autoatribuição → rota → viagem → comprovante; captura offline → sincronização).

**Rationale**: O front-end usa Vite como bundler (`evidencias`, seção 2.5); Vitest é o executor de testes nativo desse ecossistema, compartilha configuração com o Vite e evita um segundo pipeline de build para testes. React Testing Library é o padrão de fato para testar componentes React por comportamento (não por implementação), coerente com a organização por `feature` de `CONVENTIONS.md` §1.2. Playwright cobre os cenários que exigem um navegador real, essenciais para validar a operação offline (Service Worker/Workbox) e a captura de câmera/geolocalização simuladas.

**Alternatives considered**:
- **Jest** — mais comum historicamente, mas exige configuração adicional para funcionar bem com Vite/ESM; sem vantagem sobre Vitest neste projeto.
- **Cypress** para E2E — viável, mas Playwright tem suporte nativo mais maduro para simular condição offline (`context.setOffline()`), necessário para testar RF14/RNF02.

## 2. Metas operacionais (desempenho e capacidade)

**Decision**: Não adotar metas de throughput/latência formais para o piloto; adotar como *assumption* de projeto que:
- o processamento do OCR de um BDO deve concluir em poucos segundos (referência de usabilidade em campo, não meta contratual);
- a fila offline do motorista deve suportar, no mínimo, um dia inteiro de entregas de um motorista (dezenas de registros) sem perda de dados;
- os relatórios de fechamento devem ser gerados em tempo compatível com o uso interativo pelo administrador do tenant (segundos, não minutos), dado o volume mensal de BDOs limitado pelo plano de assinatura do tenant (FR-016/FR-017).

**Rationale**: O documento de evidência não define metas quantitativas de desempenho — apenas a meta de acurácia do OCR (≥ 90% dos campos obrigatórios, SC-002) e a restrição de que o processamento roda em servidor próprio, com desempenho inferior a serviços comerciais equivalentes (seção 3.10, riscos). Como o piloto atende a uma única empresa parceira com frota reduzida (seção 1.2), metas de escala de SaaS público seriam especulativas e violariam o princípio de simplicidade (`CONSTITUTION.md` §2.3).

**Alternatives considered**: Definir metas numéricas formais (ex. p95 < 200ms) foi descartado por falta de base nas evidências e por não ser um requisito do piloto — pode ser revisitado quando o sistema for validado com dados reais de uso (critério de sucesso do TAP, seção 3.13).

## 3. Isolamento multi-tenant

**Decision**: Tabelas compartilhadas com coluna discriminadora de tenant e Row Level Security (RLS) nativa do PostgreSQL, com a identidade do tenant propagada via claim no token JWT e aplicada por interceptador/filtro a cada requisição.

**Rationale**: Decisão já tomada e justificada no documento de evidência (Quadro 2, seção 2.4): menor custo por tenant, atualização única do sistema e provisionamento imediato de novos tenants (FR-001), coerentes com o público-alvo de empresas pequenas e médias sem estrutura de TI própria. Banco por tenant ou schema por tenant foram descartados na própria fonte por custo e degradação de manutenção.

**Alternatives considered**: Já registrados e descartados na fonte — ver Quadro 2 do documento de evidência. Não há necessidade de nova pesquisa.

**Boas práticas a aplicar na implementação**:
- Toda entidade de negócio carrega a coluna de tenant e uma política de RLS correspondente — nunca aplicar o filtro apenas na camada de aplicação.
- Testes automatizados de isolamento (tentativa de leitura cross-tenant) fazem parte da definição de pronto de cada história que toca em dados persistidos (FR-022).
- O filtro de tenant é resolvido uma única vez por requisição (na borda, a partir do JWT) e propagado por contexto, nunca recebido como parâmetro de entrada vindo do cliente.

## 4. Extração de dados do BDO (OCR)

**Decision**: Tesseract OCR 5 via Tess4J, com pré-processamento de imagem (binarização/deskew) e treino orientado ao layout monoespaçado do telex WorldTracer; extração sempre seguida de tela de conferência obrigatória (FR-003).

**Rationale**: Já decidido e justificado nas evidências (seção 2.5, Quadro 3): motor de código aberto, sem custo por página, executável no próprio servidor — atende à restrição de custo zero (RNF08). A conferência obrigatória (RN01 em `requisitos/03-regras-negocio.md`) é o mecanismo que absorve a acurácia inferior a um serviço comercial (risco identificado na seção 3.10).

**Alternatives considered**: Serviços comerciais de OCR (ex. Textract, Vision AI) dariam maior acurácia, mas violam a restrição de custo zero do piloto (seção 3.9) — descartados na própria fonte.

## 5. Cálculo de rota, distância e pedágios

**Decision**: Motor de rotas open source (Valhalla ou OSRM) executado localmente sobre extrato regional do OpenStreetMap, com geocodificação via Nominatim ou Photon; trechos com pedágio identificados pelo motor de rotas e cruzados com o cadastro de praças de pedágio por tenant (FR-013).

**Rationale**: Já decidido nas evidências (seção 2.5, Quadro 3) pelas mesmas razões de custo zero. A escolha entre Valhalla e OSRM fica em aberto na fonte ("Valhalla ou OSRM") — tratada aqui como decisão de implementação, não de produto.

**Alternatives considered**: Entre Valhalla e OSRM, Valhalla tem suporte mais direto a múltiplas alternativas de rota (necessário para RI03/HU03 — motorista escolhe entre alternativas), o que o torna a escolha default recomendada; OSRM permanece como alternativa caso o desempenho de Valhalla no hardware do piloto seja insatisfatório.

## 6. Operação offline e sincronização

**Decision**: Fila local em IndexedDB (via Dexie.js) no dispositivo do motorista, com Service Worker gerado por Workbox para instalação como PWA; sincronização automática, em ordem cronológica, assim que a conectividade for restabelecida (FR-020, FR-021).

**Rationale**: Já decidido nas evidências (seção 2.5, Quadro 3). O padrão de fila local + replay ordenado ao reconectar é o padrão estabelecido para operações offline-first em PWA e evita a necessidade de resolução de conflitos complexa, já que cada motorista só gera dados para as próprias entregas (sem edição concorrente do mesmo registro por dois motoristas).

**Alternatives considered**: Sincronização em tempo real via WebSocket foi descartada implicitamente pela própria natureza do requisito (RNF02 exige operação *sem* conexão); não se aplica a um cenário que precisa funcionar offline por definição.

## 7. Congelamento (snapshot) da rota

**Decision**: A viagem armazena uma cópia imutável dos valores calculados no momento do início (quilometragem de ida/volta, pedágios, tempo estimado) como um registro próprio, e não como referência a um cálculo recalculável.

**Rationale**: Requisito de negócio explícito (RN02 em `requisitos/03-regras-negocio.md`, RD04): o valor pago deve ser sempre reconstituível a partir do dado congelado, nunca de uma nova consulta. Tecnicamente, isso é um "value object" imutável persistido junto à entidade Viagem — não um ponteiro para uma tabela de tarifas ou de rotas que pode mudar depois.

**Alternatives considered**: Recalcular sob demanda a partir da tabela de tarifas/pedágios vigente foi descartado — é exatamente o comportamento que o documento de evidência aponta como causa da divergência recorrente entre empresa e motorista (seção 1.2), que o projeto existe para eliminar.

## 8. Geração de relatórios do fechamento

**Decision**: Apache PDFBox como opção default para geração dos relatórios em PDF, com JasperReports Library como alternativa caso o time de desenvolvimento precise de um designer visual de relatórios mais rico (identidade visual por tenant, FR-019).

**Rationale**: Ambas já estão pré-aprovadas nas evidências (seção 2.5, Quadro 3) e sob licenças compatíveis com a restrição de custo zero. PDFBox é mais leve e programático (bom para relatórios tabulares como o de fechamento); JasperReports facilita templates visuais reutilizáveis por tenant, relevante para RI07 (identidade visual por tenant).

**Alternatives considered**: A escolha final entre as duas pode ser adiada para a fase de implementação de HU13, sem impacto no modelo de dados ou nos contratos definidos na Fase 1.
