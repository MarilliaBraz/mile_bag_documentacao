# Requisitos

Especificação de requisitos do sistema **Milebag**, levantada a partir do documento de evidência do projeto (`evidencias/Projeto_de_software_-_ADS_com_TAP.md`), em especial das seções 1 (Introdução), 2 (Escopo do projeto) e 3.6 (Requisitos de alto nível do TAP).

Os requisitos de alto nível (RF01–RF11 e RNF01–RNF05) definidos no TAP foram detalhados e complementados com base na descrição do problema, dos objetivos do sistema e do escopo. Onde um requisito ou regra não tem número correspondente no TAP, ele foi inferido diretamente do texto do documento — a origem está indicada em cada item.

## Estrutura

```
requisitos/
├── 01-requisitos-funcionais.md      # RF — o que o sistema deve fazer
├── 02-requisitos-nao-funcionais.md  # RNF — qualidades e restrições técnicas
├── 03-regras-negocio.md             # RN — políticas e regras da operação
├── 04-regras-dominio.md             # RD — conceitos e invariantes do domínio
└── 05-regras-interface.md           # RI — regras de interação com o usuário
```

## Critério de classificação

- **Requisito funcional (RF):** uma função ou capacidade que o sistema deve executar.
- **Requisito não funcional (RNF):** uma qualidade, restrição técnica ou atributo de execução (segurança, desempenho, disponibilidade, portabilidade, documentação).
- **Regra de negócio (RN):** uma política da operação de devolução de bagagens que existe independentemente do software (ex.: cobrar apenas a quilometragem complementar da bagagem carona) e que o sistema deve fazer cumprir.
- **Regra de domínio (RD):** um conceito, invariante ou relação estrutural do modelo de domínio (ex.: o que é um tenant, o ciclo de vida de uma entrega, o que compõe o snapshot de rota).
- **Regra de interface (RI):** uma regra sobre como o usuário interage com o sistema ou sobre o comportamento exigido da interface (ex.: tela de conferência obrigatória, número mínimo de toques por entrega).

## Rastreabilidade

Cada item traz um campo **Origem**, referenciando a seção do documento de evidência (`evidencias/Projeto_de_software_-_ADS_com_TAP.md`) de onde foi extraído ou inferido. Os IDs RF01–RF11 e RNF01–RNF05 preservam a numeração original da tabela da seção 3.6; itens adicionais continuam a numeração (RF12+, RNF06+).
