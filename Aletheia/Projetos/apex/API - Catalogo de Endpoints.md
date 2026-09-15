# API Apex — Catálogo de endpoints

Voltar: [[Projetos/apex/API - Visao Geral|Visão geral]]

Legenda: **Auth** = Bearer JWT | **Público** = sem token

---

## Raiz e health

### GET /

- **Auth:** público
- **Throttle:** isento (health module pattern on health only; root uses global limit)
- **200:** `{ "message": "Hello World" }`

### GET /health

- **Auth:** público
- **Throttle:** isento
- **200:**

```json
{
  "status": "ok",
  "timestamp": "2026-09-15T18:00:00.000Z"
}
```

### GET /health/ready

- **Auth:** público
- **Throttle:** isento
- **200:** `{ "status": "ready", "timestamp": "..." }` após `SELECT 1` no Postgres
- **503/500:** falha de banco (via exceção Prisma)

---

## Auth

Ver detalhes: [[Projetos/apex/API - Autenticacao|Autenticação]]

| Método | Path | Auth | Status |
|--------|------|------|--------|
| POST | `/api/v1/auth/token` | Público | 200 |
| POST | `/api/v1/auth/login` | Público | 200 |
| POST | `/api/v1/auth/refresh` | Público | 200 |
| POST | `/api/v1/auth/logout` | Público | 204 |

---

## Erros de integração

Base: `/api/v1/errors`

### POST /api/v1/errors/ingest

Ingestão em lote (snapshot por `integration` + `source`).

- **Auth:** Bearer (service ou user)
- **Rate limit:** 20/min
- **Status:** **201**

#### Request body

```json
{
  "integration": "TASY",
  "source": "servidor-rp-01",
  "resolveAbsent": true,
  "errors": [
    {
      "message": "Falha na integração",
      "errorCode": "mapeamento",
      "filePath": "/var/log/tasy.log",
      "occurredAt": "2026-06-15T10:30:00Z",
      "metadata": { "modulo": "pacientes" }
    }
  ]
}
```

| Campo | Obrigatório | Regras |
|-------|-------------|--------|
| `integration` | sim | `TASY` \| `MV` |
| `source` | sim | string, máx. 255 |
| `errors` | sim | array, máx. **500** itens |
| `resolveAbsent` | não | boolean, default **true** |

**Item `errors[]`:**

| Campo | Obrigatório | Regras |
|-------|-------------|--------|
| `message` | sim | máx. 2000 |
| `errorCode` | não | máx. 100 |
| `filePath` | não | máx. 500 |
| `occurredAt` | não | ISO 8601 date string |
| `metadata` | não | objeto JSON, máx. **4096 bytes** serializado |

#### Response 201

```json
{
  "received": 1,
  "new": 1,
  "updated": 0,
  "reactivated": 0,
  "resolved": 0,
  "notified": 1
}
```

#### Erros comuns

- **400** — validação DTO (campos extras rejeitados: `forbidNonWhitelisted`)
- **401** — token ausente/inválido

---

### GET /api/v1/errors

Lista paginada por cursor.

- **Auth:** Bearer
- **Status:** 200

#### Query params

| Param | Tipo | Descrição |
|-------|------|-----------|
| `integration` | `TASY` \| `MV` | filtro |
| `status` | `ACTIVE` \| `RESOLVED` | filtro |
| `source` | string | filtro, máx. 255 |
| `limit` | int | default **50**, máx. **100** |
| `cursor` | string | ID do último item da página anterior |

Ordenação: `lastSeenAt desc`, `id desc`.

#### Response 200

```json
{
  "data": [ "/* IntegrationError */" ],
  "nextCursor": "clxxx..." ,
  "hasMore": true
}
```

Cada item em `data` segue o modelo [[Projetos/apex/API - Modelos e Enums#IntegrationError|IntegrationError]].

---

### GET /api/v1/errors/:id

Detalhe de um erro.

- **Auth:** Bearer
- **Status:** 200 — objeto `IntegrationError`
- **404** — `Error not found`

---

## Signum (MSSQL)

Formato de resposta: `{ "error": boolean, "message": string, "data?": ... }`.

401 nestes endpoints:

```json
{ "error": true, "message": "Unauthorized" }
```

### GET /api/v1/status/fila/tasy

Fila Tasy (resultados).

- **Auth:** Bearer
- **200:** `{ "error": false, "message": "Sucesso", "data": [ FilaResultadoType ] }`
- **400/500:** `{ "error": true, "message": "..." }`

**FilaResultadoType:** `rcl_pac`, `rcl_cod`, `rcl_osm_serie`, `rcl_osm`, `rcl_dthr_lib`, `obs`

### GET /api/v1/status/fila/checkup

Mesmo contrato; fila Checkup (resultados).

### GET /api/v1/status/fila/interface

- **200 data:** array de `FilaInterfaceType`: `FRS_ID`, `FRS_ELB_COD`, `FRS_AMOSTRA`, `FRS_DTHR_REG`, `FRS_STATUS`

### GET /api/v1/consulta/exame/:smk_cod

Detalhe de exame pelo código SMK.

- **Auth:** Bearer
- **400** — `smk_cod` vazio ou erro de negócio MSSQL
- **200 data (Exame):**

```json
{
  "smk_cod": "EX001",
  "smk_nome": "Hemograma",
  "smk_status": "A",
  "str_nome": "Laboratório",
  "amostras": [{ "amo_cod": "", "amo_qlf_rot": "", "cod_exm_tasy": "" }],
  "atributos": [{ "dic_dsc": "", "atr_num": "", "atr_rot": "", "atr_tipo": "" }]
}
```

---

## Admin — service clients

Base: `/api/v1/admin/service-clients`

- **Auth:** Bearer **ADMIN** (`typ: user`, `role: ADMIN`)
- Service JWT → **403** `Insufficient permissions`

### POST /

Cria client; **secret retornado uma vez**.

#### Request

```json
{
  "clientId": "monitor-hp-prod",
  "name": "Monitor HP produção"
}
```

| Campo | Regras |
|-------|--------|
| `clientId` | máx. 100, regex `^[a-zA-Z0-9._-]+$` |
| `name` | máx. 255 |

#### Response 201

```json
{
  "id": "cuid",
  "clientId": "monitor-hp-prod",
  "name": "Monitor HP produção",
  "isActive": true,
  "createdAt": "...",
  "updatedAt": "...",
  "clientSecret": "<mostrar-e-guardar>"
}
```

- **409** — `clientId already exists`

### GET /

Lista clients (sem secret).

### GET /:id

Detalhe (sem secret).

- **404** — `Service client not found`

### PATCH /:id

```json
{ "name": "Novo nome", "isActive": false }
```

Ambos opcionais.

### POST /:id/rotate-secret

Novo secret (retornado **uma vez** no body).

---

## Admin — usuários

Base: `/api/v1/admin/users`

- **Auth:** ADMIN

### POST /

```json
{
  "email": "user@example.com",
  "password": "senha-min-8",
  "name": "Opcional",
  "role": "USER"
}
```

| Campo | Regras |
|-------|--------|
| `email` | e-mail |
| `password` | 8–128 chars |
| `name` | opcional, máx. 255 |
| `role` | opcional, `ADMIN` \| `USER`, default `USER` |

Response: user sem password. **409** — `email already exists`

### GET / — listar

### GET /:id — detalhe

- **404** — `User not found`

### PATCH /:id

```json
{ "name": "...", "role": "ADMIN", "isActive": true }
```

Regras de auto-edição:

- Admin **não** pode alterar **próprio** `role`
- Admin **não** pode se **desativar** (`isActive: false`)

Desativar usuário revoga refresh tokens.

- **400** — mensagens acima

### POST /:id/reset-password

```json
{ "password": "nova-senha-8+" }
```

---

## Admin — sessões

Base: `/api/v1/admin/sessions`

### GET /?userId=

- **Query `userId`:** obrigatório (string)
- **200:** array de sessões:

```json
[
  {
    "id": "...",
    "userId": "...",
    "createdAt": "...",
    "expiresAt": "...",
    "revokedAt": null
  }
]
```

- **404** — `User not found`

### DELETE /:id

Revoga sessão (refresh token). Idempotente se já revogada.

- **404** — `Session not found`

### DELETE /by-user/:userId

- **200:** `{ "revokedCount": 3 }`
- **404** — `User not found`

---

## Resumo rápido (tabela)

| Método | Path | Quem |
|--------|------|------|
| GET | `/`, `/health`, `/health/ready` | Público |
| POST | `/api/v1/auth/*` | Público |
| POST | `/api/v1/errors/ingest` | Bearer |
| GET | `/api/v1/errors`, `/api/v1/errors/:id` | Bearer |
| GET | `/api/v1/status/fila/*` | Bearer |
| GET | `/api/v1/consulta/exame/:smk_cod` | Bearer |
| * | `/api/v1/admin/*` | ADMIN |
