# Histórias de Usuário (HUs)

Histórias de usuário do sistema **Milebag**, derivadas dos requisitos funcionais listados em `../requisitos/01-requisitos-funcionais.md`. Cada HU é um arquivo próprio e referencia o(s) requisito(s) funcional(is) de origem, com critérios de aceite que incorporam as regras de negócio (RN), de domínio (RD), de interface (RI) e os requisitos não funcionais (RNF) aplicáveis, definidos em `../requisitos/`.

Requisitos puramente automáticos ou de infraestrutura sem interação direta de um usuário (ex.: RF04 — congelamento da rota) foram incorporados como critério de aceite dentro da HU do fluxo que os dispara, em vez de constituírem uma HU própria.

## Estrutura

```
HUs/
├── motorista/
│   ├── HU01-capturar-bdo-por-fotografia.md
│   ├── HU02-autoatribuir-entrega.md
│   ├── HU03-calcular-e-escolher-rota.md
│   ├── HU04-iniciar-viagem-rota-congelada.md
│   ├── HU05-vincular-bagagem-carona.md
│   ├── HU06-registrar-comprovante-entrega.md
│   ├── HU07-registrar-ocorrencia-nao-entrega.md
│   └── HU08-operar-offline-sincronizar.md
├── administrador-tenant/
│   ├── HU09-configurar-tabelas-tarifa.md
│   ├── HU10-cadastrar-pracas-pedagio.md
│   ├── HU11-parametrizar-configuracoes-tenant.md
│   ├── HU12-gerar-fechamento-quinzenal.md
│   └── HU13-emitir-relatorios-identidade-visual.md
└── administrador-plataforma/
    ├── HU14-cadastrar-empresa-assinante.md
    ├── HU15-convidar-primeiro-administrador-tenant.md
    └── HU16-gerenciar-planos-assinatura.md
```

## Formato

Cada HU segue o formato:

> **Como** \<persona\>,
> **eu quero** \<ação\>,
> **para que** \<benefício\>.

seguido de **Requisito(s) relacionado(s)** e **Critérios de aceite**.

## Índice

| HU | Título | Persona | Requisito(s) | Arquivo |
|---|---|---|---|---|
| HU01 | Capturar o BDO por fotografia | Motorista | RF01 | `motorista/HU01-capturar-bdo-por-fotografia.md` |
| HU02 | Autoatribuir a entrega | Motorista | RF02 | `motorista/HU02-autoatribuir-entrega.md` |
| HU03 | Calcular e escolher a rota da entrega | Motorista | RF03 | `motorista/HU03-calcular-e-escolher-rota.md` |
| HU04 | Iniciar a viagem com a rota congelada | Motorista | RF04 | `motorista/HU04-iniciar-viagem-rota-congelada.md` |
| HU05 | Vincular bagagem carona à entrega | Motorista | RF05 | `motorista/HU05-vincular-bagagem-carona.md` |
| HU06 | Registrar o comprovante de entrega | Motorista | RF06 | `motorista/HU06-registrar-comprovante-entrega.md` |
| HU07 | Registrar ocorrência de não entrega | Motorista | RF07 | `motorista/HU07-registrar-ocorrencia-nao-entrega.md` |
| HU08 | Operar offline e sincronizar automaticamente | Motorista | RF14, RNF02 | `motorista/HU08-operar-offline-sincronizar.md` |
| HU09 | Configurar tabelas de tarifa | Administrador do tenant | RF08 | `administrador-tenant/HU09-configurar-tabelas-tarifa.md` |
| HU10 | Cadastrar praças de pedágio | Administrador do tenant | RF11 | `administrador-tenant/HU10-cadastrar-pracas-pedagio.md` |
| HU11 | Parametrizar configurações do tenant | Administrador do tenant | RF13 | `administrador-tenant/HU11-parametrizar-configuracoes-tenant.md` |
| HU12 | Gerar o fechamento quinzenal | Administrador do tenant | RF09 | `administrador-tenant/HU12-gerar-fechamento-quinzenal.md` |
| HU13 | Emitir relatórios com identidade visual do tenant | Administrador do tenant | RF15 | `administrador-tenant/HU13-emitir-relatorios-identidade-visual.md` |
| HU14 | Cadastrar empresa assinante (tenant) | Administrador da plataforma | RF10 | `administrador-plataforma/HU14-cadastrar-empresa-assinante.md` |
| HU15 | Convidar o primeiro administrador do tenant | Administrador da plataforma | RF10 | `administrador-plataforma/HU15-convidar-primeiro-administrador-tenant.md` |
| HU16 | Gerenciar planos de assinatura | Administrador da plataforma | RF12 | `administrador-plataforma/HU16-gerenciar-planos-assinatura.md` |
