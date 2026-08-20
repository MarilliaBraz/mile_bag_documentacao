# Fase 0 — Pesquisa e Decisões Técnicas: HU13

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Nenhuma decisão técnica nova é necessária além do que já está registrado em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção **8. Geração de relatórios do fechamento**, que já cobre a escolha entre PDFBox e JasperReports e recomenda JasperReports quando um template visual por tenant for necessário — exatamente o caso desta HU.

Ponto específico não coberto na Fase 0 do plano-mãe:

## Formato e limites do arquivo de logotipo

**Decision**: Aceitar upload de imagem raster (PNG ou JPEG), com limite de tamanho de arquivo e dimensão máxima definidos na validação de upload (mesmo padrão já usado para as imagens de BDO e de comprovante de entrega).

**Rationale**: Reaproveita a validação de upload de imagem já necessária para HU01 (captura do BDO) e HU06 (comprovante de entrega), evitando um novo mecanismo específico só para logotipo.

**Alternatives considered**: Aceitar SVG foi descartado por simplicidade — exigiria sanitização adicional (SVG pode conter script embutido) sem benefício claro para um logotipo estático em relatório PDF.
