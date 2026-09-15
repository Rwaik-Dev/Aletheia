# API Apex — Autenticação e autorização

Voltar: [[Projetos/apex/API - Visao Geral|Visão geral]]

## JWT — claims

Payload assinado (HMAC com `JWT_SECRET`):

| Claim | Service client | Usuário |
|-------|----------------|---------|
| `sub` | ID do `ServiceClient` | ID do `User` |
| `typ` | `"service"` | `"user"` |
| `clientId` | string | — |
| `email` | — | string |
| `role` | — | `"ADMIN"` \| `"USER"` |

Validação: guard global `JwtAuthGuard` em todas as rotas exceto `@Public()`.

Admin: `@Roles('ADMIN')` + `RolesGuard` — exige `typ === 'user'` e `role === 'ADMIN'`.

---

## POST /api/v1/auth/token

Emite access token para **service clients** (scripts).

- **Auth:** pública
- **Rate limit:** 20 req/min (além do global 100/min)
- **Status:** 200

### Request

```json
{
  "clientId": "apex-ingest-client",
  "clientSecret": "<secret>"
}
```

| Campo | Regras |
|-------|--------|
| `clientId` | string, não vazio |
| `clientSecret` | string, não vazio |

### Response 200

```json
{
  "accessToken": "<jwt>",
  "expiresIn": 900
}
```

### Erros

- **401** `Invalid client credentials` — client inexistente, inativo ou secret incorreto

### Notas para clients

- Não há refresh token para service clients: **reautentique** antes de `expiresIn` (margem ~30 s recomendada).
- Provisionamento de clients: admin `POST /api/v1/admin/service-clients` (secret retornado uma vez).

---

## POST /api/v1/auth/login

Login de **usuário humano**.

- **Auth:** pública
- **Rate limit:** 20/min
- **Status:** 200

### Request

```json
{
  "email": "admin@example.com",
  "password": "senha-min-8-chars"
}
```

| Campo | Regras |
|-------|--------|
| `email` | e-mail válido |
| `password` | string, mín. **8** caracteres |

### Response 200

```json
{
  "accessToken": "<jwt>",
  "refreshToken": "<opaco>",
  "expiresIn": 900
}
```

### Erros

- **401** `Invalid email or password`

---

## POST /api/v1/auth/refresh

Renova tokens de usuário (**rotação** de refresh).

- **Auth:** pública
- **Rate limit:** 20/min
- **Status:** 200

### Request

```json
{
  "refreshToken": "<opaco>"
}
```

### Response 200

Mesmo shape do login (novo `accessToken` + novo `refreshToken`). O refresh antigo é **revogado**.

### Erros

- **401** `Invalid or expired refresh token`

---

## POST /api/v1/auth/logout

Revoga refresh token.

- **Auth:** pública
- **Status:** **204** (sem body)

### Request

```json
{
  "refreshToken": "<opaco>"
}
```

Token já revogado ou desconhecido: **204** (idempotente).

---

## Sessões (refresh tokens no banco)

Cada login/refresh cria registro em `RefreshToken` (hash opaco, `expiresAt`, `revokedAt`).

Admin pode listar/revogar: [[Projetos/apex/API - Catalogo de Endpoints#Admin — sessões|Admin — sessões]].

Desativar usuário (`isActive: false`) revoga todos os refresh tokens ativos.

---

## CORS (somente browser)

Variável `CORS_ORIGINS` (vírgulas). Exemplo local:

```env
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
```

- `credentials: true`
- Headers permitidos: `Content-Type`, `Authorization`, `X-Request-Id`
- Métodos: GET, HEAD, PUT, PATCH, POST, DELETE, OPTIONS

Service clients server-side **não** usam CORS.

---

## Bootstrap inicial

`prisma db seed` com `BOOTSTRAP_CLIENT_*` e `BOOTSTRAP_ADMIN_*` cria client e admin iniciais (ver Runbook).
