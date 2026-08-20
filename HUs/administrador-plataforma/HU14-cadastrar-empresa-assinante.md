# HU14 — Cadastrar empresa assinante (tenant)

**Persona:** Administrador da plataforma (provedor)

**Como** administrador da plataforma,
**eu quero** cadastrar uma nova empresa de entrega como tenant,
**para que** ela possa começar a operar no sistema sem que eu precise implantar uma instância dedicada para ela.

**Requisito(s) relacionado(s):** RF10

**Critérios de aceite:**
- O cadastro deve criar o registro do tenant, associado a um plano de assinatura (HU16, RD11).
- Nenhuma etapa de instalação ou configuração de infraestrutura deve ser necessária para o novo tenant (RN09, RNF10).
- Os dados do novo tenant devem ficar isolados de todos os demais tenants desde o primeiro registro (RNF01).
