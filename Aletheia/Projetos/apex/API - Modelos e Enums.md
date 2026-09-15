# API Apex — Modelos, enums e fingerprint

Voltar: [[Projetos/apex/API - Visao Geral|Visão geral]]

## Enums

### Integration

Valores aceitos na ingestão:

- `TASY`
- `MV`
- `HP` (notificação igual MV: uma vez por erro; webhook `GOOGLE_CHAT_WEBHOOK_HP`)

### ErrorStatus

- `ACTIVE` — erro ainda presente no snapshot do client
- `RESOLVED` — ausente no snapshot (quando `resolveAbsent: true`) ou resolvido manualmente pelo fluxo

### UserRole

- `ADMIN`
- `USER`

---

## IntegrationError

Entidade persistida (Postgres). Campos expostos nas APIs de listagem/detalhe:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | string (cuid) | PK |
| `integration` | Integration | |
| `source` | string | identificador do emissor (servidor, host) |
| `fingerprint` | string | SHA-256 hex (ver abaixo) |
| `message` | string | |
| `errorCode` | string? | ex. `mapeamento`, `tabela de preço` |
| `filePath` | string? | |
| `occurredAt` | datetime? | |
| `metadata` | JSON? | |
| `status` | ErrorStatus | |
| `firstSeenAt` | datetime | |
| `lastSeenAt` | datetime | atualizado a cada ingest |
| `resolvedAt` | datetime? | |
| `lastNotifiedAt` | datetime? | |
| `notificationCount` | int | |
| `createdAt` | datetime | |
| `updatedAt` | datetime | |

**Unicidade:** `(integration, source, fingerprint)`

---

## Fingerprint (deduplicação)

Função: `buildFingerprint` — SHA-256 de:

```text
{integration}|{source}|{message.trim()}|{errorCode ?? ""}|{filePath ?? ""}
```

Implicações para clients:

- Mesma mensagem + mesmo `errorCode` + mesmo `filePath` na mesma `source` → **mesmo erro** (update/reactivate, não duplicata).
- Alterar `message` ou códigos gera **novo** erro.
- `metadata` **não** entra no fingerprint.

---

## IngestionLog

Registro agregado por request de ingest (auditoria interna):

`integration`, `source`, `receivedCount`, `newCount`, `resolvedCount`, `updatedCount`, `reactivatedCount`, `notifiedCount`, `createdAt`

Não exposto via API REST pública atualmente.

---

## ServiceClient (admin)

Campos listados: `id`, `clientId`, `name`, `isActive`, `createdAt`, `updatedAt`.

`clientSecret` só no create e rotate-secret.

Secret armazenado como hash (bcrypt).

---

## User (admin)

Campos: `id`, `email`, `name`, `role`, `isActive`, `createdAt`, `updatedAt`.

Senha nunca retornada.

---

## RefreshToken / sessão

Campos expostos ao admin: `id`, `userId`, `createdAt`, `expiresAt`, `revokedAt`.

TTL refresh: `JWT_REFRESH_TTL_SECONDS` (default **604800** = 7 dias).

---

## Signum — tipos em `data`

Ver [[Projetos/apex/API - Catalogo de Endpoints#Signum (MSSQL)|Catálogo Signum]]:

- `FilaResultadoType`
- `FilaInterfaceType`
- `Exame` + `Amostra` + `Atributo`

---

## Notificações (contexto para ingest clients)

Regras completas: [[Projetos/apex/Regras de Negocio|Regras de negócio]].

Resumo:

| Integração | Notifica quando |
|------------|-----------------|
| TASY | `errorCode` ∈ `mapeamento`, `tabela de preço` |
| MV | uma vez por erro novo |

`errorCode` Tasy fora dessa lista: persiste, mas **não** dispara Google Chat.

---

## Variáveis de ambiente relevantes para clients

| Variável | Efeito |
|----------|--------|
| `JWT_ACCESS_TTL_SECONDS` | `expiresIn` do token (mín. 60) |
| `JWT_REFRESH_TTL_SECONDS` | vida do refresh (mín. 3600) |
| `TASY_REMINDER_INTERVAL_HOURS` | re-notificação Tasy (default 2) |

Clients só precisam conhecer TTL do access token na prática; demais é operação do servidor.
