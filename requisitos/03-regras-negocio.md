# Regras de Negócio (RN)

Políticas da operação de devolução de bagagens que existem independentemente do software e que o sistema deve fazer cumprir.

## RN01 — Conferência obrigatória dos dados extraídos do BDO

**Descrição:** O reconhecimento automático (OCR) apenas sugere os valores dos campos do BDO; o operador deve confirmá-los antes que o registro seja considerado válido. Nenhum BDO pode ser processado com base apenas na extração automática, sem conferência humana.

**Origem:** Seção 2.2.1.

## RN02 — Pagamento calculado sobre o snapshot congelado da rota

**Descrição:** O valor pago ao motorista deve ser calculado exclusivamente a partir do snapshot de rota (quilometragem, pedágios, tempo) registrado no início da viagem, nunca de uma nova consulta ao serviço de mapas feita posteriormente. O valor deve ser sempre reconstituível a partir desse dado congelado.

**Origem:** Seções 2.2.2 e 3.4.

## RN03 — Cobrança de bagagem carona limitada à quilometragem complementar

**Descrição:** Quando uma bagagem adicional (carona) é vinculada a uma entrega principal, a remuneração do motorista considera apenas a quilometragem complementar decorrente dessa bagagem, e não uma nova viagem completa.

**Origem:** Seções 2.5 (Quadro 3) e 3.6 (RF05).

## RN04 — Entrega só se encerra com comprovante registrado

**Descrição:** O ciclo de uma entrega não pode ser considerado concluído sem o registro do comprovante de entrega (POD), contendo foto do documento assinado, identificação do recebedor, data, hora e geolocalização.

**Origem:** Seção 2.2.4.

## RN05 — Ocorrência de não entrega reabre o BDO

**Descrição:** Quando a entrega não se concretiza, deve ser registrada uma ocorrência padronizada e o BDO correspondente deve ser reaberto para nova tentativa, não podendo permanecer em estado de finalizado sem entrega comprovada.

**Origem:** Seções 3.5 e 3.6 (RF07).

## RN06 — Pedágio sempre cobrado em ida e volta

**Descrição:** O valor de reembolso de pedágio é sempre calculado considerando o trajeto de ida e de volta, com base nas praças cadastradas e em vigor na data da viagem.

**Origem:** Seção 3.6 (RF11).

## RN07 — Tarifas parametrizáveis por tenant, motorista e região

**Descrição:** O valor pago por entrega é determinado por tabela de tarifas configurável, podendo variar por tenant, por motorista específico e por região, combinando valores fixos e valores por quilômetro.

**Origem:** Seções 2.4 e 3.6 (RF08).

## RN08 — Fechamento financeiro periódico e segregado

**Descrição:** A apuração de valores devidos aos motoristas é feita em fechamentos quinzenais, com relatórios de pagamento de entregas e de reembolso de pedágios gerados separadamente, sem necessidade de ajuste manual.

**Origem:** Seções 3.5 e 3.13.

## RN09 — Provisionamento de tenant não exige implantação dedicada

**Descrição:** A entrada de uma nova empresa assinante no sistema é feita apenas pela criação do registro do tenant e convite ao primeiro administrador — não pode depender de instalação, configuração de infraestrutura ou intervenção técnica dedicada por cliente.

**Origem:** Seção 2.4.

## RN10 — Planos de assinatura diferenciados por volume de uso

**Descrição:** A comercialização do serviço segue planos de assinatura diferenciados pelo número de motoristas ativos e pelo volume mensal de BDOs processados pelo tenant.

**Origem:** Seção 2.4.

## RN11 — Evolução do sistema aplicada uniformemente a todos os tenants

**Descrição:** Atualizações e correções do sistema devem ser aplicadas de uma só vez a todos os tenants (base compartilhada), não sendo permitido manter versões divergentes por cliente.

**Origem:** Seção 2.4.

## RN12 — Exclusões de escopo da operação

**Descrição:** As seguintes atividades não fazem parte da operação suportada pelo sistema e não devem ser tratadas como responsabilidade dele: integração eletrônica direta com sistemas das companhias aéreas; emissão de documentos fiscais; cobrança automática de assinaturas via gateway de pagamento; roteirização automática com otimização de múltiplas paradas; atendimento ao passageiro final; e tratamento de indenizações por extravio definitivo.

**Origem:** Seções 2.6 e 3.5 (fora do escopo).
