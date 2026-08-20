# Requisitos Funcionais (RF)

## RF01 — Capturar imagem do BDO

**Descrição:** O sistema deve permitir o registro do BDO por fotografia, com extração automática dos campos do padrão WorldTracer por OCR (Tesseract) e tela de conferência obrigatória, na qual o reconhecimento sugere os valores e o operador confirma. A imagem original deve ser anexada ao registro como prova documental.

**Origem:** Seções 2.2.1 e 3.6 (RF01).

## RF02 — Autoatribuir a entrega

**Descrição:** O motorista deve poder atribuir a si mesmo a entrega no momento da captura fotográfica do BDO.

**Origem:** Seções 2.2.3 e 3.6 (RF02).

## RF03 — Calcular a rota da entrega

**Descrição:** O sistema deve calcular distância, tempo estimado e trechos com pedágio por meio de serviço de rotas de código aberto (Valhalla ou OSRM) sobre dados do OpenStreetMap, permitindo ao motorista escolher entre rotas alternativas sugeridas ou traçar rota própria.

**Origem:** Seções 2.2.3 e 3.6 (RF03).

## RF04 — Congelar a quilometragem da viagem

**Descrição:** O sistema deve registrar, no início da viagem, um snapshot da rota escolhida (quilometragem de ida e volta, praças de pedágio e tempo estimado), mantendo esses valores congelados como base do pagamento, independentemente de consultas futuras ao serviço de mapas.

**Origem:** Seções 2.2.2 e 3.6 (RF04).

## RF05 — Vincular bagagem carona

**Descrição:** O sistema deve permitir vincular bagagens adicionais a uma entrega principal, remunerando apenas a quilometragem complementar decorrente da bagagem carona.

**Origem:** Seção 3.6 (RF05).

## RF06 — Registrar o comprovante de entrega (POD)

**Descrição:** O sistema deve registrar o comprovante de entrega com foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas, disponibilizando esse conjunto para prestação de contas à companhia aérea. O ciclo da entrega só se encerra mediante esse registro.

**Origem:** Seções 2.2.4 e 3.6 (RF06).

## RF07 — Registrar ocorrências de entrega

**Descrição:** O sistema deve permitir registrar ocorrências padronizadas quando a entrega não se concretizar, reabrindo o BDO correspondente.

**Origem:** Seção 3.6 (RF07).

## RF08 — Configurar tabelas de tarifa

**Descrição:** O sistema deve permitir configurar tarifas por tenant, por motorista e por região, incluindo valores fixos e valores por quilômetro.

**Origem:** Seção 3.6 (RF08).

## RF09 — Gerar o fechamento quinzenal

**Descrição:** O sistema deve emitir relatórios periódicos (quinzenais) de pagamento de entregas e de reembolso de pedágios, separadamente, sem necessidade de ajuste manual.

**Origem:** Seções 3.6 (RF09) e 3.13 (critérios de sucesso).

## RF10 — Gerenciar empresas assinantes

**Descrição:** O sistema deve permitir o cadastro de empresas assinantes (tenants), planos de assinatura e usuários, com convite ao primeiro administrador do tenant no momento do provisionamento.

**Origem:** Seções 2.4 e 3.6 (RF10).

## RF11 — Cadastrar praças de pedágio

**Descrição:** O sistema deve permitir o cadastro das praças de pedágio e de seus valores por tenant, aplicados sempre em ida e volta, com registro da data de vigência de cada valor.

**Origem:** Seções 3.6 (RF11) e 3.10 (riscos, mitigação "registrar data de vigência").

## RF12 — Cadastrar e gerenciar planos de assinatura

**Descrição:** O sistema deve permitir cadastrar planos de assinatura diferenciados por número de motoristas ativos e por volume mensal de BDOs processados, associando cada tenant a um plano.

**Origem:** Seção 2.4.

## RF13 — Parametrizar configurações por tenant

**Descrição:** O sistema deve permitir que cada tenant configure, de forma independente: tabelas de tarifa, motivos de ocorrência, campos obrigatórios no formulário de baixa, bases aeroportuárias, companhias aéreas atendidas, periodicidade do fechamento e identidade visual dos relatórios.

**Origem:** Seção 2.4.

## RF14 — Sincronizar dados capturados offline

**Descrição:** O sistema deve sincronizar automaticamente, assim que a conectividade for restabelecida, os dados capturados pela aplicação do motorista durante a operação sem conexão (BDOs, rotas, comprovantes e ocorrências).

**Origem:** Seções 2.5 e 3.5 (escopo incluído — "aplicação do motorista em modo offline, com sincronização automática").

## RF15 — Emitir relatórios com identidade visual por tenant

**Descrição:** O sistema deve gerar os relatórios do fechamento periódico (PDF) com a identidade visual configurada por cada tenant.

**Origem:** Seções 2.4 e 2.5 (Quadro 3, linha "Relatórios").
