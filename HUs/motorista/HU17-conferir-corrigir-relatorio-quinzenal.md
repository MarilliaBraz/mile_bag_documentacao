# HU17 — Conferir e corrigir meu relatório quinzenal

**Persona:** Motorista

**Como** motorista,
**eu quero** visualizar, imprimir e corrigir (com justificativa) os dados usados no cálculo do meu pagamento em cada entrega do período,
**para que** eu possa conferir o valor antes do fechamento e resolver divergências sem depender de contato manual com a empresa.

**Requisito(s) relacionado(s):** RF16, RN13, RD13, RI11

**Critérios de aceite:**
- Devo poder visualizar e imprimir um relatório com todas as minhas entregas do período de fechamento em andamento, incluindo os valores usados no cálculo do pagamento de cada uma (RI11).
- Para uma entrega específica, devo poder corrigir diretamente a quilometragem, a tarifa aplicada ou o valor de pedágio usado no cálculo (RD13).
- Não devo conseguir salvar nenhuma correção sem preencher uma justificativa (RI11, RN13).
- O valor originalmente congelado na viagem deve continuar visível e preservado, mesmo depois de eu registrar um ajuste — a correção não apaga o dado original (RD13, RN02).
- O fechamento gerado pelo administrador do tenant deve usar o valor mais recente (meu ajuste, se eu tiver feito algum) no cálculo do pagamento (RN13).
