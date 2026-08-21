# Regras de Domínio (RD)

Conceitos, invariantes e relações estruturais do modelo de domínio do sistema.

## RD01 — Tenant

**Descrição:** Um tenant é uma empresa terceirizada de entrega domiciliar de bagagens. Cada tenant possui, de forma isolada, seus próprios motoristas, bases aeroportuárias, contratos com companhias aéreas e tabelas de tarifa. Todos os tenants são atendidos por uma única instância da aplicação e um único banco de dados, distinguidos por uma coluna discriminadora de propriedade presente nas tabelas do domínio.

**Origem:** Seção 2.4.

## RD02 — BDO (Baggage Delivery Order)

**Descrição:** O BDO é o telex do WorldTracer, impresso pela companhia aérea, que autoriza e formaliza a devolução de uma bagagem irregular ao passageiro. É o documento de origem de toda entrega no sistema — nenhuma entrega existe sem um BDO capturado. Um BDO capturado permanece associado à imagem original como prova documental.

**Origem:** Seções 1.2 e 2.2.1.

## RD03 — Entrega principal e bagagem carona

**Descrição:** Uma entrega principal pode ter uma ou mais bagagens carona vinculadas a ela. A bagagem carona não constitui uma entrega independente para fins de remuneração — apenas adiciona quilometragem complementar à entrega principal à qual está vinculada.

**Origem:** Seções 2.2 e 3.6 (RF05).

## RD04 — Snapshot de rota (viagem)

**Descrição:** No início de cada viagem, é registrado um snapshot imutável composto por: quilometragem de ida, quilometragem de volta, praças de pedágio do trajeto e tempo estimado. Esse snapshot, uma vez criado, é o dado de referência único para o cálculo do pagamento — não é substituído por novas consultas ao serviço de mapas.

**Origem:** Seções 2.2.2 e 3.6 (RF04).

## RD05 — Ciclo de vida da entrega

**Descrição:** Uma entrega percorre os seguintes estados: BDO capturado → autoatribuído a um motorista → rota calculada e snapshot congelado → em execução → encerrada com comprovante de entrega (POD) OU com ocorrência registrada e BDO reaberto (retornando ao início do ciclo). Uma entrega não transita para "encerrada com sucesso" sem POD registrado.

**Origem:** Seções 2.2.1 a 2.2.4 e 3.6 (RF01–RF07).

## RD06 — Comprovante de entrega (POD)

**Descrição:** O comprovante de entrega é composto por: foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas. É o registro que evidencia, para fins de prestação de contas à companhia aérea, que a entrega foi concluída.

**Origem:** Seção 2.2.4.

## RD07 — Ocorrência

**Descrição:** Uma ocorrência é um evento padronizado (de um catálogo configurável por tenant) registrado quando uma entrega não se concretiza. Toda ocorrência está associada a uma entrega e provoca a reabertura do BDO correspondente.

**Origem:** Seções 2.4 e 3.6 (RF07).

## RD08 — Praça de pedágio

**Descrição:** Uma praça de pedágio é um cadastro por tenant, com valor associado e data de vigência. Os trechos com pedágio identificados numa rota são associados às praças cadastradas para compor o snapshot de rota e o cálculo de reembolso.

**Origem:** Seções 2.5 (Quadro 3) e 3.10 (riscos).

## RD09 — Tabela de tarifa

**Descrição:** Uma tabela de tarifa define valores (fixos e por quilômetro) aplicáveis a uma combinação de tenant, motorista e região. É a base de cálculo do valor devido ao motorista por entrega, no fechamento periódico.

**Origem:** Seções 2.4 e 3.6 (RF08).

## RD10 — Fechamento (período de pagamento)

**Descrição:** Um fechamento agrega, para um período quinzenal, os valores devidos a cada motorista por entregas realizadas e o reembolso de pedágios, tratados como totais separados. O fechamento consome os snapshots de rota e as tabelas de tarifa vigentes no período.

**Origem:** Seções 3.5 e 3.13.

## RD11 — Plano de assinatura

**Descrição:** Um plano de assinatura delimita, por tenant, o número de motoristas ativos permitidos e o volume mensal de BDOs processáveis. Todo tenant está associado a exatamente um plano vigente.

**Origem:** Seção 2.4.

## RD12 — Perfis de usuário

**Descrição:** O domínio reconhece, no mínimo, os perfis de administrador do tenant, operador de back-office e motorista, cada um com autorizações distintas sobre as funcionalidades do sistema, propagadas via claim de tenant e perfil no token de autenticação.

**Origem:** Seções 2.5 e 3.6 (RNF03).

## RD13 — Ajuste de viagem

**Descrição:** Um ajuste é um registro que corrige um dos valores calculados de uma viagem já congelada (quilometragem de ida/volta, tarifa aplicada ou valor de pedágio), feito pelo próprio motorista, sempre com justificativa obrigatória e data/hora do ajuste. O ajuste nunca sobrescreve o snapshot original da viagem (RD04) — ele é um registro adicional, vinculado à viagem, que passa a ser o valor usado no fechamento (RN13) enquanto o valor congelado original permanece preservado no histórico, para auditoria.

**Origem:** Definida em sessão de trabalho de 2026-08-21, a pedido da administração/product owner (RN13, RF16).
