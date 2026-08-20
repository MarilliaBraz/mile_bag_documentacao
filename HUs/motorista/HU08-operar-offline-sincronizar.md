# HU08 — Operar offline e sincronizar automaticamente

**Persona:** Motorista

**Como** motorista,
**eu quero** continuar capturando BDOs, rotas, comprovantes e ocorrências mesmo sem conexão com a internet,
**para que** interrupções de rede durante a rota não interrompam meu trabalho.

**Requisito(s) relacionado(s):** RF14, RNF02

**Critérios de aceite:**
- Todas as ações de captura (BDO, rota, POD, ocorrência) devem funcionar normalmente sem conexão, armazenando os dados localmente no dispositivo.
- A interface deve indicar visivelmente quando estou operando offline e quais registros estão pendentes de sincronização (RI05).
- Assim que a conectividade for restabelecida, os dados pendentes devem ser sincronizados automaticamente, sem ação manual de minha parte.
