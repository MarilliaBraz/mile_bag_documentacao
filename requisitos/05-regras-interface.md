# Regras de Interface (RI)

Regras sobre como o usuário interage com o sistema ou sobre o comportamento exigido da interface, em cada aplicação (back-office e aplicação do motorista).

## RI01 — Tela de conferência obrigatória após OCR

**Descrição:** Após a extração automática dos campos do BDO por OCR, o sistema deve apresentar uma tela de conferência exibindo os valores sugeridos, exigindo confirmação (ou correção) do operador antes de permitir o avanço do fluxo. Não deve existir caminho que salve o registro diretamente a partir do OCR sem essa etapa.

**Origem:** Seção 2.2.1.

## RI02 — Interface do motorista otimizada para uso em campo

**Descrição:** A interface da aplicação do motorista deve ser projetada para uso em smartphone, durante a rota, priorizando o mínimo de toques possível para concluir cada etapa da entrega (autoatribuição, escolha de rota, comprovante, ocorrência).

**Origem:** Seção 3.6 (RNF04).

## RI03 — Escolha de rota entre alternativas ou traçado próprio

**Descrição:** Ao calcular a rota de uma entrega, a interface deve apresentar ao motorista as rotas alternativas sugeridas pelo serviço de mapas e permitir, como opção explícita, o traçado de uma rota própria diferente das sugeridas.

**Origem:** Seção 2.2.3.

## RI04 — Aplicação do motorista como PWA instalável

**Descrição:** A aplicação do motorista deve ser entregue como aplicação web progressiva (PWA), instalável no dispositivo, com acesso à câmera (captura de BDO e de comprovante) e ao GPS (geolocalização), e com armazenamento local para operação offline.

**Origem:** Seção 2.5 (Quadro 3).

## RI05 — Indicação visível de operação offline e status de sincronização

**Descrição:** Quando a aplicação do motorista estiver operando sem conexão, a interface deve indicar esse estado ao usuário e sinalizar quando os dados capturados estiverem pendentes de sincronização ou já sincronizados.

**Origem:** Seções 2.5 e 3.6 (RNF02).

## RI06 — Back-office responsivo para administrador do tenant e provedor

**Descrição:** A aplicação de back-office (React/TypeScript) deve ser responsiva, atendendo tanto ao administrador do tenant quanto ao provedor da plataforma, com tipagem estática aplicada às regras de exibição de valores monetários e de tarifas.

**Origem:** Seção 2.5 (Quadro 3).

## RI07 — Relatórios com identidade visual por tenant

**Descrição:** As telas de configuração do back-office devem permitir que cada tenant defina sua identidade visual (ex.: logotipo, cabeçalho) e essa identidade deve refletir-se nos relatórios periódicos emitidos em PDF.

**Origem:** Seções 2.4 e 2.5 (Quadro 3).

## RI08 — Atribuição visível do OpenStreetMap nas telas de mapa

**Descrição:** Toda tela que exibir um mapa construído sobre dados do OpenStreetMap deve apresentar, de forma visível, a atribuição exigida pela licença ODbL.

**Origem:** Seção 3.11 (recursos e tecnologias, "licenciamento").

## RI09 — Onboarding de tenant por convite ao primeiro administrador

**Descrição:** O fluxo de cadastro de uma nova empresa assinante deve, ao final do provisionamento do tenant, disparar um convite (por e-mail ou mecanismo equivalente) ao primeiro administrador, sem exigir etapas manuais adicionais de configuração para que o tenant comece a operar.

**Origem:** Seção 2.4.

## RI10 — Formulário de baixa com campos obrigatórios configuráveis por tenant

**Descrição:** A interface de registro de ocorrência ou de encerramento de entrega ("baixa") deve exibir campos obrigatórios que variam conforme a configuração definida por cada tenant, e não um formulário fixo único para todos os clientes.

**Origem:** Seção 2.4.
