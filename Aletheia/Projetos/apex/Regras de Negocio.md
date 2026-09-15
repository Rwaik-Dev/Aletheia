# Regras de Negocio - Apex

## Ingestão de erros

- Endpoint: `POST /api/v1/errors/ingest`
- Auth: `Authorization: Bearer <access_token>` (service client ou usuário)
- Integrações: `TASY` | `MV`
- Até **500** erros por requisição
- `resolveAbsent` (default `true`): erros ativos ausentes no payload são marcados **RESOLVED**
- Cada `source` deve enviar o **snapshot completo** dos erros ativos daquela integração
- Deduplicação: fingerprint estável por `(integration, source, fingerprint)`

### Campos por erro

- `message` (obrigatório, máx. 2000)
- `errorCode`, `filePath`, `occurredAt`, `metadata` (opcionais; limites no DTO)

## Notificação Google Chat (Tasy)

Só notifica quando `errorCode` é:

- `mapeamento`
- `tabela de preço` (webhook dedicado `GOOGLE_CHAT_WEBHOOK_TASY_TABELA_PRECO`)

Comportamento:

- Novo ou reativado: notifica
- Persistente: re-notifica a cada `TASY_REMINDER_INTERVAL_HOURS` (padrão 2h); job roda a cada **15 min**
- Resolvido: para de notificar

## Notificação Google Chat (MV)

- Notifica **uma vez** por erro (novo)
- Reativação ou persistência: **não** re-notifica

## Monitoramento de filas (NTFY)

- Cron: a cada **5 minutos**
- Limite: **40** itens em qualquer fila (Tasy resultado, Checkup resultado, Interface)
- Se alguma fila falhar na consulta MSSQL, o job aborta aquele ciclo (sem alerta parcial)

## Autorização

| Ator | Autenticação | Acesso |
|------|--------------|--------|
| Service client | `POST /api/v1/auth/token` | Ingest, erros, signum; **sem** `/admin/*` |
| User ADMIN | login + JWT com `role: ADMIN` | Tudo, incluindo admin |
| User USER | login | Consultas; **403** em `/admin/*` |

Rotas públicas: health, `GET /`, `/api/v1/auth/*`.

## Respostas Signum

Endpoints em `/api/v1/status/fila/*` e `/api/v1/consulta/exame/:smk_cod` retornam `{ error, message, data? }`. Erro de auth nesses endpoints usa `error: true` no body.
