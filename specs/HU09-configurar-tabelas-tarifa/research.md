# Fase 0 — Pesquisa e Decisões Técnicas: HU09

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

Nenhuma decisão técnica nova além do que já está registrado em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md). Em especial, aplica-se diretamente a seção **"3. Isolamento multi-tenant"** daquele documento (RLS + coluna discriminadora de tenant, propagada via claim JWT) — a tabela de tarifa é mais uma entidade sujeita a essa mesma política, sem particularidade própria.

A única decisão específica desta HU é de modelagem, não de tecnologia, e está documentada em [data-model.md](./data-model.md): a resolução de qual tarifa se aplica a uma entrega (motorista > região > padrão) é lógica de negócio pura (`service`), não uma consulta com prioridade no banco — evita depender de ordenação implícita de linhas.
