# API Apex — Erros HTTP, rate limit e headers

Voltar: [[Projetos/apex/API - Visao Geral|Visão geral]]

## Headers

### Request (recomendados)

```http
Content-Type: application/json
Authorization: Bearer <accessToken>
```

Opcional: clients podem enviar `X-Request-Id`; se omitido, o servidor gera UUID.

### Response

- `X-Request-Id` — ID de correlação (middleware global)

---

## Rate limiting

`ThrottlerGuard` global:

- **100 requisições / minuto / IP** (janela 60 s)

Overrides (mesma janela, limite próprio):

| Rotas | Limite |
|-------|--------|
| `POST /api/v1/auth/token`, `login`, `refresh` | 20/min |
| `POST /api/v1/errors/ingest` | 20/min |

Isento de throttle:

- `GET /health`
- `GET /health/ready`

**429** quando excedido (Nest Throttler).

---

## Body size

JSON parser: **1 MB** máximo (`express.json({ limit: '1mb' })`).

---

## Validação (400)

`ValidationPipe` global:

- `whitelist: true` — remove campos desconhecidos
- `forbidNonWhitelisted: true` — **400** se enviar propriedade extra
- `transform: true` — coerção de tipos (query `limit` numérico, etc.)

Corpo típico Nest 400:

```json
{
  "statusCode": 400,
  "message": ["email must be an email"],
  "error": "Bad Request"
}
```

---

## Autenticação

### REST padrão (erros, admin, auth)

| Status | Situação | Body `message` (exemplos) |
|--------|----------|---------------------------|
| **401** | Sem Bearer / JWT inválido | `Invalid or missing access token` |
| **401** | Login/token/refresh | `Invalid client credentials`, `Invalid email or password`, `Invalid or expired refresh token` |
| **403** | Admin sem role | `Insufficient permissions` |
| **404** | Recurso | `Error not found`, `User not found`, etc. |
| **409** | Conflito | `clientId already exists`, `email already exists` |

### Signum (`/api/v1/status/fila/*`, `/api/v1/consulta/exame/*`)

Filter dedicado preserva shape:

```json
{ "error": true, "message": "Unauthorized" }
```

Outros erros Signum (400/500):

```json
{ "error": true, "message": "Falha ao consultar a fila Tasy, consultar os logs!" }
```

Sucesso Signum:

```json
{ "error": false, "message": "Sucesso", "data": [] }
```

---

## Prisma / banco

`PrismaExceptionFilter` global traduz erros conhecidos do Prisma para HTTP (ex.: conflitos).

---

## Helmet

Headers de segurança HTTP via Helmet (`crossOriginResourcePolicy: cross-origin` para compatibilidade com frontend).

---

## Checklist de debug para integradores

1. **401 REST** — token expirado? usar `/auth/token` ou refresh (usuários)
2. **401 Signum com `{ error: true }`** — mesmo diagnóstico, formato diferente
3. **403 admin** — token de service client ou user `USER`
4. **400 ingest** — enum `integration`, tamanho de arrays/strings, `metadata` > 4096 bytes
5. **429** — backoff; ingest em lotes ≤ 500 já ajuda
6. **CORS** (browser) — origin exata em `CORS_ORIGINS`, não confundir porta 3000 (UI) com 3333 (API)
