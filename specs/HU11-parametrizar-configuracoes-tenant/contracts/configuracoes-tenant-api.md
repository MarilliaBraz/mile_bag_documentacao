# Contrato de API: Configurações do Tenant (HU11)

**Autenticação**: token com claim de tenant; escrita exigida perfil administrador do tenant; leitura também permitida à aplicação do motorista (motivos de ocorrência e campos obrigatórios da baixa).

## Motivos de ocorrência

### `GET /api/v1/configuracoes/motivos-ocorrencia`
Lista os motivos ativos do tenant autenticado (usado pelo app do motorista, HU07).

### `POST /api/v1/configuracoes/motivos-ocorrencia`
- **Request**: `descricao`.
- **Response 201**.

### `PATCH /api/v1/configuracoes/motivos-ocorrencia/{id}`
- **Request**: `{ ativo: boolean }` (FR-009 — nunca `DELETE`).
- **Response 200**.

## Campos obrigatórios do formulário de baixa

### `GET /api/v1/configuracoes/campos-baixa`
Lista os campos disponíveis e sua obrigatoriedade para o tenant (usado pelo app do motorista, HU06/HU07).

### `PUT /api/v1/configuracoes/campos-baixa`
- **Request**: lista de `{ campo, obrigatorio }`.
- **Response 200**.
- **Erros**: `400` — `campo` fora do catálogo fixo do sistema.

## Bases aeroportuárias

### `GET /api/v1/configuracoes/bases-aeroportuarias`
### `POST /api/v1/configuracoes/bases-aeroportuarias`
- **Request**: `codigoIata`, `nome`.
- **Response 201**.

### `DELETE /api/v1/configuracoes/bases-aeroportuarias/{id}`
- **Response 204**.

## Companhias aéreas atendidas

### `GET /api/v1/configuracoes/companhias-aereas`
### `POST /api/v1/configuracoes/companhias-aereas`
- **Request**: `codigoIata`, `nome`.
- **Response 201**.

### `DELETE /api/v1/configuracoes/companhias-aereas/{id}`
- **Response 204**.

## Periodicidade de fechamento

### `GET /api/v1/configuracoes/periodicidade-fechamento`
Retorna a periodicidade vigente e a próxima, se já houver uma agendada.

### `POST /api/v1/configuracoes/periodicidade-fechamento`
- **Request**: `periodicidade`, `vigenteDesde` (MUST ser posterior ao fim do período em andamento — FR-008).
- **Response 201**.
- **Erros**: `400` — `vigenteDesde` dentro do período de fechamento já em andamento.
