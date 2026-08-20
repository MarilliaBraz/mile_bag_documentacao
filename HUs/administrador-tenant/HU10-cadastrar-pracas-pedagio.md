# HU10 — Cadastrar praças de pedágio

**Persona:** Administrador do tenant / Back-office

**Como** administrador do tenant,
**eu quero** cadastrar as praças de pedágio e seus valores,
**para que** o reembolso de pedágio pago aos motoristas seja calculado corretamente.

**Requisito(s) relacionado(s):** RF11

**Critérios de aceite:**
- Devo poder cadastrar uma praça de pedágio com valor e data de vigência (RD08).
- O cálculo de reembolso deve sempre considerar o trajeto de ida e de volta (RN06).
- Ao atualizar o valor de uma praça, o valor anterior deve ser preservado com sua data de vigência, para não alterar viagens já congeladas (RN02).
