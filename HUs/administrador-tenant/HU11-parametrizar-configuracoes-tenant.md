# HU11 — Parametrizar configurações do tenant

**Persona:** Administrador do tenant / Back-office

**Como** administrador do tenant,
**eu quero** configurar os motivos de ocorrência, os campos obrigatórios do formulário de baixa, as bases aeroportuárias, as companhias aéreas atendidas e a periodicidade do fechamento,
**para que** o sistema se adapte à forma como minha empresa opera, sem depender do fornecedor do sistema.

**Requisito(s) relacionado(s):** RF13

**Critérios de aceite:**
- Cada uma dessas configurações deve ser exclusiva do meu tenant, sem afetar ou ser afetada por outros tenants (RNF01).
- O catálogo de ocorrências configurado aqui deve ser o mesmo exibido ao motorista ao registrar uma ocorrência (RD07, HU07).
- Os campos obrigatórios configurados devem refletir-se na interface de baixa exibida ao motorista (RI10).
