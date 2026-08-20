# HU04 — Iniciar a viagem com a rota congelada

**Persona:** Motorista

**Como** motorista,
**eu quero** iniciar a viagem com a rota escolhida,
**para que** a quilometragem e os pedágios usados no meu pagamento fiquem garantidos, independentemente de qualquer consulta futura ao mapa.

**Requisito(s) relacionado(s):** RF04

**Critérios de aceite:**
- Ao iniciar a viagem, o sistema deve registrar um snapshot com quilometragem de ida e volta, praças de pedágio e tempo estimado (RD04).
- Esse snapshot não pode ser alterado por uma nova consulta ao serviço de mapas após o início da viagem (RN02).
- O valor a ser pago pela entrega deve ser sempre reconstituível a partir desse snapshot.
