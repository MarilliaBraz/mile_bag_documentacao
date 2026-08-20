# Implementation Plan: Milebag — Histórias de Usuário da Operação de Última Milha

**Branch**: `001-milebag-user-stories` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `mile_bag_documentacao/specs/001-milebag-user-stories/spec.md`

## Summary

Sistema web SaaS multi-tenant que digitaliza a última milha da devolução de bagagens extraviadas: captura do BDO por foto com OCR assistido, cálculo e congelamento de rota, comprovante de entrega geolocalizado, tratamento de bagagem carona e ocorrências, motor de tarifas e pedágios parametrizável por tenant, fechamento financeiro periódico e onboarding de novas empresas assinantes sem implantação dedicada — tudo operável offline pelo motorista em campo, com isolamento estrito de dados entre tenants.

Abordagem técnica: back-end único (Java/Spring Boot 3) compartilhado por todos os tenants, com isolamento de dados por Row Level Security no PostgreSQL; duas aplicações de front-end em React/TypeScript (back-office administrativo e PWA do motorista, com fila offline em IndexedDB); serviços de OCR (Tesseract) e de rotas (Valhalla/OSRM) executados localmente, sem dependência de APIs comerciais pagas — consistente com a restrição de custo zero do piloto (documento de evidência, seção 3.9).

## Technical Context

**Language/Version**: Java 21 (LTS) no back-end; TypeScript com React 18+ no front-end (back-office e PWA do motorista)

**Primary Dependencies**: Spring Boot 3 (Spring MVC, Spring Data JPA/Hibernate, Spring Security com JWT), Flyway, Tess4J (Tesseract OCR 5), cliente para serviço de rotas open source (Valhalla ou OSRM) e geocodificação (Nominatim ou Photon), Apache PDFBox ou JasperReports Library, Springdoc OpenAPI; front-end: React + Vite, Workbox (service worker PWA), Dexie.js (IndexedDB), Leaflet sobre camadas OpenStreetMap

**Storage**: PostgreSQL com extensão PostGIS; isolamento multi-tenant por tabelas compartilhadas com coluna discriminadora de tenant e Row Level Security nativa; imagens de BDO e de comprovante em volume de disco com prefixo por tenant, atrás de uma interface de repositório (migrável para armazenamento compatível com S3, ex. SeaweedFS); fila local em IndexedDB no dispositivo do motorista para a operação offline

**Testing**: JUnit 5, Mockito e Testcontainers (contra instância real de PostgreSQL) no back-end; testes de contrato de API com Bruno/Hoppscotch. Framework de teste do front-end React não especificado no documento de evidência — resolvido em `research.md` (Fase 0)

**Target Platform**: back-end, banco de dados e serviços de OCR/rotas em contêineres Docker/Podman, hospedados em servidor próprio da empresa parceira durante o piloto (sem nuvem); back-office acessado via navegador desktop/mobile; aplicação do motorista como PWA instalável em smartphone (Android/iOS via navegador)

**Project Type**: web — aplicação SaaS multi-tenant composta por 1 backend de API e 2 front-ends (back-office e PWA do motorista), distribuídos em repositórios separados (ver Project Structure)

**Performance Goals**: sem metas de throughput/latência quantitativas nas evidências, exceto a acurácia mínima de 90% na extração automática dos campos obrigatórios do BDO (SC-002 do spec). Metas operacionais adicionais (tempo de resposta percebido, tempo de processamento do OCR) resolvidas como *assumptions* em `research.md` (Fase 0), por ausência de exigência explícita e por se tratar de piloto de baixa escala

**Constraints**: operação offline obrigatória na aplicação do motorista, com sincronização automática (FR-020, FR-021); isolamento total de dados entre tenants, verificado por testes automatizados (FR-022); autenticação por token com claim de tenant e tratamento de dados pessoais do passageiro conforme a LGPD; toda a pilha tecnológica deve ser software livre/código aberto, sem custo de licença ou de nuvem no piloto; atribuição visível do OpenStreetMap em toda tela de mapa

**Scale/Scope**: piloto com uma empresa parceira real; volume de motoristas ativos e de BDOs mensais delimitado pelo plano de assinatura de cada tenant (FR-016, FR-017), não por uma meta absoluta de usuários simultâneos

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

O arquivo `.specify/memory/constitution.md` (raiz de `Projeto_PI`) está com os placeholders do template, não preenchido. A constituição e as convenções técnicas efetivamente adotadas pelo projeto vivem no repositório irmão `mile_bag_audite` (`CONSTITUTION.md` e `CONVENTIONS.md`), referenciado como tal pelos READMEs de `mile_bag_api` e `mile_bag_app`. Os gates abaixo usam esses documentos.

**Nota de inconsistência (não bloqueante)**: a seção "Missão/Escopo" de `mile_bag_audite/CONSTITUTION.md` descreve um produto de *auditoria de bagagens vinculadas a programas de milhas*, anterior ao pivô documentado no TAP (`evidencias/Projeto_de_software_-_ADS_com_TAP.md`) para a gestão da última milha de devolução de bagagens extraviadas. Os princípios gerais (rastreabilidade, auditabilidade, simplicidade, segurança/privacidade, transparência) e as convenções técnicas (`CONVENTIONS.md`) são agnósticos a esse pivô e continuam válidos — só a narrativa de missão/escopo está desatualizada. Recomenda-se atualizar essa seção em uma sessão futura; isso não bloqueia este plano.

| Gate | Fonte | Avaliação |
|---|---|---|
| Simplicidade antes de generalidade | `CONSTITUTION.md` §2.3 | PASS — estrutura por domínio (tenant, BDO, entrega, rota, POD, ocorrência, tarifa, pedágio, fechamento, plano) sem camadas especulativas |
| Confiabilidade e rastreabilidade dos dados | `CONSTITUTION.md` §2.1 | PASS — snapshot imutável de rota (FR-007) e histórico de ocorrências preservado (FR-011) já fazem parte do spec |
| Segurança e privacidade por padrão | `CONSTITUTION.md` §2.4 | PASS — isolamento entre tenants (FR-022) e conformidade LGPD são requisitos não funcionais do spec |
| Separação `core`/`business` (backend) | `CONVENTIONS.md` §1.1 | PASS — ver Project Structure |
| Organização por `feature` (frontend) | `CONVENTIONS.md` §1.2 | PASS — ver Project Structure |
| Os 3 pilares: parametrização, ambiente, versão | `CONVENTIONS.md` §2.3 | PASS — tarifas, pedágios, motivos de ocorrência e campos obrigatórios são parametrizáveis por tenant (FR-012, FR-013, FR-018), nunca hardcoded; profiles de ambiente e migrações Flyway incrementais a adotar na implementação |
| Autoria de commits sempre humana | `CONSTITUTION.md` §4 | Aplica-se à condução do projeto, não ao desenho técnico — sem impacto neste plano |

Nenhuma violação identificada. Seção **Complexity Tracking** não se aplica.

## Project Structure

### Documentation (this feature)

```text
mile_bag_documentacao/specs/001-milebag-user-stories/
├── spec.md               # Especificação (/speckit-specify command output)
├── plan.md               # Este arquivo (/speckit-plan command output)
├── research.md           # Saída da Fase 0 (/speckit-plan command)
├── data-model.md         # Saída da Fase 1 (/speckit-plan command)
├── quickstart.md         # Saída da Fase 1 (/speckit-plan command)
├── contracts/            # Saída da Fase 1 (/speckit-plan command)
└── tasks.md              # Saída da Fase 2 (/speckit-tasks command — NÃO criado por /speckit-plan)
```

### Source Code (multi-repositório)

O projeto Milebag é versionado em repositórios Git separados sob `Projeto_PI/`, e não como pastas de um monorepo único. A feature descrita em `spec.md` é implementada através de três desses repositórios:

```text
mile_bag_api/                              # Back-end — Java 21 / Spring Boot 3
├── src/main/java/com/marillia/milebag/
│   ├── core/                              # infraestrutura genérica, sem regra de negócio
│   │   ├── config/                        # security (JWT), contexto de tenant, beans
│   │   ├── exception/                     # tratamento padronizado de erros
│   │   ├── util/
│   │   └── base/                          # BaseEntity, BaseRepository, BaseService
│   ├── business/                          # regras de negócio, um subpacote por domínio
│   │   ├── tenant/                        # tenant, plano de assinatura, onboarding (US1, US2)
│   │   ├── bdo/                           # captura, OCR, conferência (US1)
│   │   ├── entrega/                       # ciclo de vida da entrega, bagagem carona (US1, US2)
│   │   ├── rota/                          # cálculo de rota e snapshot de viagem (US1)
│   │   ├── comprovante/                   # POD — comprovante de entrega (US1)
│   │   ├── ocorrencia/                    # ocorrências e reabertura de BDO (US2)
│   │   ├── tarifa/                        # tabelas de tarifa (US2)
│   │   ├── pedagio/                       # praças de pedágio (US2)
│   │   ├── fechamento/                    # fechamento periódico (US2)
│   │   └── configuracao/                  # parametrização por tenant, identidade visual (US3)
│   └── MilebagApplication.java
├── src/main/resources/
│   ├── db/migration/                      # migrações Flyway, incrementais
│   ├── application.yml
│   ├── application-dev.yml
│   └── application-prod.yml
└── src/test/java/...                      # espelha src/main/java (JUnit5, Mockito, Testcontainers)

mile_bag_app/                              # Front-end — React + TypeScript + Vite
├── src/
│   ├── features/
│   │   ├── back-office/
│   │   │   ├── tenant-onboarding/         # US1 (HU14, HU15)
│   │   │   ├── planos/                    # US2 (HU16)
│   │   │   ├── tarifas/                   # US2 (HU09)
│   │   │   ├── pedagios/                  # US2 (HU10)
│   │   │   ├── fechamento/                # US2 (HU12, HU13)
│   │   │   └── configuracao-tenant/       # US3 (HU11)
│   │   └── motorista/                     # PWA
│   │       ├── captura-bdo/               # US1 (HU01, HU02)
│   │       ├── rota/                      # US1 (HU03, HU04)
│   │       ├── bagagem-carona/            # US2 (HU05)
│   │       ├── comprovante-entrega/       # US1 (HU06)
│   │       ├── ocorrencia/                # US2 (HU07)
│   │       └── offline-sync/              # US1 (HU08) — fila IndexedDB (Dexie.js) + Workbox
│   ├── shared/                            # usado por 2+ features (auth, mapas/Leaflet, api client)
│   ├── App.tsx
│   └── main.tsx
└── public/manifest.json                   # PWA installable

mile_bag_infra/                            # Contêineres e implantação
├── docker-compose.yml                     # api, front-end, PostgreSQL+PostGIS, serviço de rotas, proxy reverso
└── services/
    ├── ocr/                               # Tesseract OCR 5, treinado para o telex WorldTracer
    └── routing/                           # Valhalla ou OSRM sobre extrato OpenStreetMap regional
```

**Structure Decision**: Aplicação web multi-tenant com backend único e dois front-ends (Option 2 do template, adaptada para multi-repositório). O back-end (`mile_bag_api`) segue a separação `core`/`business` de `CONVENTIONS.md` §1.1, com um subpacote `business` por domínio da entrega (tenant, bdo, entrega, rota, comprovante, ocorrência, tarifa, pedágio, fechamento, configuração) — mapeados às 3 User Stories do spec. O front-end (`mile_bag_app`) segue a organização por `feature` de `CONVENTIONS.md` §1.2, separando as features do back-office das features do motorista (PWA), cada uma referenciando a(s) HU(s) de origem. A infraestrutura (`mile_bag_infra`) isola os serviços de OCR e de rotas como contêineres próprios, alinhado à restrição de custo zero (sem serviços comerciais pagos).

## Complexity Tracking

*Não aplicável — nenhuma violação de gate identificada na Constitution Check.*
