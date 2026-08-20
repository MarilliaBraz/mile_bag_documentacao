# HU06 — Registrar o comprovante de entrega

**Persona:** Motorista

**Como** motorista,
**eu quero** registrar o comprovante de entrega ao concluir a devolução da bagagem,
**para que** a entrega seja considerada oficialmente concluída e a empresa tenha prova para a companhia aérea.

**Requisito(s) relacionado(s):** RF06

**Critérios de aceite:**
- O registro deve incluir foto do documento assinado, identificação do recebedor, data, hora e coordenadas geográficas (RD06).
- A entrega só muda para o estado "encerrada com sucesso" após esse registro completo (RN04, RD05).
- Sem conexão, devo poder registrar o comprovante normalmente e tê-lo sincronizado depois (ver HU08).
