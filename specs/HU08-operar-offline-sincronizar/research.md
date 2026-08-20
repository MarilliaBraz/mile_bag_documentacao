# Fase 0 — Pesquisa e Decisões Técnicas

**Feature**: [spec.md](./spec.md) | **Plano**: [plan.md](./plan.md)

A escolha de IndexedDB/Dexie.js e Workbox já está registrada em [`../001-milebag-user-stories/research.md`](../001-milebag-user-stories/research.md), seção 6. Esta HU aprofunda o desenho do mecanismo de fila e do contrato de sincronização, que a seção 6 do research-mãe não detalha.

## 1. Identidade e idempotência dos eventos offline

**Decision**: Cada ação capturada offline recebe, no momento da criação local, um identificador único gerado no próprio dispositivo (`idLocal`, UUID v4). Esse identificador é enviado ao servidor junto ao evento e usado como chave de idempotência: se o servidor já processou um `idLocal`, uma nova submissão do mesmo evento é reconhecida e ignorada (sem erro, sem duplicação), em vez de criar um novo registro.

**Rationale**: Reenvios são inevitáveis em conectividade instável (queda no meio do envio, timeout sem confirmação recebida pelo cliente mesmo que o servidor tenha processado). Gerar o identificador no cliente — não no servidor — é o que torna o reenvio seguro (FR-006): o servidor consegue reconhecer "já vi isso" mesmo que a resposta da primeira tentativa nunca tenha chegado de volta ao dispositivo.

**Alternatives considered**: Deduplicar por conteúdo (hash dos campos do evento) foi descartado — dois eventos legítimos e distintos podem ter o mesmo conteúdo (ex.: duas ocorrências iguais em entregas diferentes no mesmo segundo), gerando falsos positivos de deduplicação.

## 2. Ordenação da sincronização

**Decision**: Cada evento carrega um `timestampCriacaoLocal` (relógio do dispositivo) e um número sequencial local (`sequenciaLocal`, incrementado a cada novo evento enfileirado no mesmo dispositivo). O lote de sincronização é enviado — e processado no servidor — respeitando `sequenciaLocal`, não a ordem de chegada da requisição.

**Rationale**: FR-005 exige preservar a ordem cronológica real dos eventos, mas o relógio do dispositivo não é uma fonte confiável isoladamente (pode estar dessincronizado, ver Edge Cases do spec). Um contador sequencial local, monotônico por dispositivo, garante ordem relativa correta mesmo que o `timestampCriacaoLocal` esteja impreciso; o timestamp continua sendo armazenado como dado de negócio (ex.: hora do comprovante de entrega), mas não é a fonte de verdade da ordem de processamento.

**Alternatives considered**: Usar apenas o timestamp do dispositivo como critério de ordenação foi descartado pelo risco de relógio incorreto citado no Edge Cases do spec; usar a ordem de chegada ao servidor foi descartado porque não reflete a ordem real dos fatos quando há reenvio parcial de um lote.

## 3. Formato do lote de sincronização

**Decision**: Um único endpoint de sincronização em lote (`POST /api/v1/sync/eventos`, ver `contracts/`) recebe um array de eventos heterogêneos (BDO capturado, entrega autoatribuída, rota escolhida, viagem iniciada, comprovante registrado, ocorrência registrada), cada um com seu `tipoEvento`, `idLocal`, `sequenciaLocal` e `payload` específico do domínio. O servidor processa o lote em ordem e devolve, para cada item, o resultado individual (aceito, duplicado, erro).

**Rationale**: Um endpoint por tipo de evento multiplicaria chamadas de rede em um cenário de conectividade ruim (exatamente o cenário que esta HU existe para mitigar); um único endpoint de lote reduz o número de round-trips e permite processar tudo o que foi acumulado offline em uma única reconexão. A resposta item a item é necessária porque um lote pode ter, por exemplo, o 3º item duplicado (já sincronizado em uma tentativa anterior) e os demais novos.

**Alternatives considered**: Endpoints separados por domínio (`POST /entregas`, `POST /ocorrencias`, etc.) chamados sequencialmente pelo cliente foram descartados por esse motivo — o objetivo aqui é minimizar chamadas de rede, não reaproveitar os contratos individuais de HU01–HU07 tal como são.

## 4. Falha parcial de lote

**Decision**: O processamento de um lote é "melhor esforço item a item", não transacional: uma falha em um item não impede o processamento dos demais itens do mesmo lote. O cliente reenvia, em uma tentativa futura, apenas os itens que retornaram erro (identificados por `idLocal`).

**Rationale**: Trata diretamente o edge case "conexão cai no meio do envio de um lote" — se o lote fosse tudo-ou-nada, uma falha em um único item (ex.: um `motivoOcorrenciaId` inválido) bloquearia a sincronização de todos os demais eventos, legítimos, do mesmo motorista.

**Alternatives considered**: Processamento transacional do lote inteiro foi descartado pelo motivo acima — reduziria a confiabilidade exatamente no cenário de rede ruim que a HU precisa suportar.
