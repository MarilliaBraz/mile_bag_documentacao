# Requisitos Não Funcionais (RNF)

## RNF01 — Isolar os dados entre tenants

**Descrição:** Os dados de uma empresa (tenant) não podem ser acessíveis a outra. O isolamento deve ser imposto em nível de linha no banco de dados (Row Level Security), ativado a partir da identidade do tenant contida no token de autenticação. É requisito não funcional crítico, verificado por testes automatizados de vazamento entre tenants a cada incremento.

**Origem:** Seções 2.4 e 3.6 (RNF01).

## RNF02 — Operar sem conexão

**Descrição:** A aplicação do motorista deve operar sem conexão à internet (modo offline), armazenando localmente os dados capturados e sincronizando automaticamente quando a rede for restabelecida.

**Origem:** Seções 2.5 e 3.6 (RNF02).

## RNF03 — Proteger o acesso e os dados pessoais

**Descrição:** O sistema deve autenticar por token (JWT contendo claim de tenant), autorizar por perfil de usuário e tratar os dados pessoais do passageiro (nome, endereço) em conformidade com a LGPD.

**Origem:** Seções 2.5, 3.6 (RNF03) e 3.9 (restrições).

## RNF04 — Ser utilizável em campo pelo motorista

**Descrição:** A interface do motorista deve ser utilizável em smartphone, durante a rota, com o mínimo de toques possível por entrega.

**Origem:** Seção 3.6 (RNF04).

## RNF05 — Documentar a interface de programação

**Descrição:** A API do sistema deve ser documentada em padrão OpenAPI, gerada a partir do código (Springdoc).

**Origem:** Seções 2.5 e 3.6 (RNF05).

## RNF06 — Manter acurácia mínima de extração do OCR

**Descrição:** A extração automática dos campos do BDO deve atingir corretamente, no mínimo, 90% dos campos obrigatórios em amostra de documentos reais, mitigado por pré-tratamento de imagem e treino do Tesseract com amostras do telex WorldTracer.

**Origem:** Seções 3.10 (riscos) e 3.13 (critérios de sucesso).

## RNF07 — Ser implantável de forma reproduzível

**Descrição:** A aplicação deve ser implantada em contêineres (Docker/Podman), garantindo ambiente padronizado e implantação reprodutível em qualquer máquina, incluindo capacidade de recuperação em caso de indisponibilidade do servidor hospedeiro.

**Origem:** Seções 2.5 e 3.10 (riscos, mitigação "contêineres... implantação reprodutível... e backup diário").

## RNF08 — Operar sem custo de licenciamento

**Descrição:** Toda a pilha tecnológica utilizada no piloto deve ser composta exclusivamente por software livre e de código aberto, sem contratação de serviços pagos ou licenças comerciais.

**Origem:** Seções 2.5, 3.9 (restrições) e 3.12 (orçamento).

## RNF09 — Exibir atribuição dos dados do OpenStreetMap

**Descrição:** As telas que exibem mapas construídos a partir de dados do OpenStreetMap devem exibir atribuição visível, conforme exigido pela licença ODbL.

**Origem:** Seções 3.9 (restrições) e 3.11 (recursos e tecnologias, "licenciamento").

## RNF10 — Provisionar novos tenants sem implantação dedicada

**Descrição:** O provisionamento de uma nova empresa assinante não deve exigir implantação de infraestrutura própria — apenas a criação do registro do tenant e o convite ao primeiro administrador, permitindo início de operação em poucas horas.

**Origem:** Seções 1.4 e 2.4.
