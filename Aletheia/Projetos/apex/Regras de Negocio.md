# Regras de Negocio - Apex

## Ingestão de erros

- Endpoint: `POST /api/v1/errors/ingest`
- Auth: `Authorization: Bearer <access_token>` (service client ou usuário)
- Integrações: `TASY` | `MV` | `HP`
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

## Notificação Google Chat (MV e HP)

- Notifica **uma vez** por erro (novo)
- Reativação ou persistência: **não** re-notifica
- HP usa webhook `GOOGLE_CHAT_WEBHOOK_HP`
- Mensagem genérica (paciente, exame, arquivo, datas) para códigos HP **diferentes** de `HP_CONN_001`

### HP_CONN_001 — divergência de layout (Hermes Pardini)

Quando `integration = HP` e `errorCode = HP_CONN_001`, o Apex **consulta o RP (MSSQL)** antes de montar o Google Chat, usando o mnemônico Pardini enviado no metadata (`exm_pardini`).

**Metadata esperado no ingest** (enviado pelo Monitor HP):

| Campo | Uso |
|-------|-----|
| `pedido` | Identificador do pedido HP |
| `chave_local` | Fingerprint local do Monitor (dedup 24h no cliente) |
| `exm_pardini` | Mnemônico do exame no HP (ex.: `A-HIP`) — **chave da consulta RP** |
| `arquivo_origem` | Nome/caminho do arquivo HTML de log |
| `cod_formato_certo` | Layout esperado no cadastro médico |
| `cod_formato_errado` | Layout encontrado no arquivo |

**Query RP** (`MssqlService.consultaExamesHpardiniPorMnemonico`):

```sql
SELECT RTRIM(smk_cod) AS cod_exame, RTRIM(smk_nome) AS nome_exame
FROM exm_hpardini, ams, smk
WHERE num_exm = ams_hpardini_num_exm
  AND ams_smk_cod = smk_cod
  AND smk_status = 'A'
  AND mn_exa = @mn_exa
```

- Pode retornar **várias linhas** (um mnemônico Pardini amarrado a mais de um exame no RP).
- Na mensagem, cada linha vira: `• Mnemônico: {cod_exame} — {nome_exame}`.

**Fallback (sempre envia notificação):**

- RP indisponível ou erro na query → aviso no bloco de detalhes + `Mnemônico HP: {exm_pardini}`.
- Nenhum exame encontrado → aviso explícito + mnemônico HP.

**Modelo da mensagem Google Chat** (título `🚨 NOVO ERRO DE IMPORTAÇÃO IDENTIFICADO`):

- Tipo `HP_CONN_001`
- Texto fixo do problema (layout informado ≠ layout cadastrado)
- Detalhes do exame (lista de mnemônicos RP ou fallback)
- Códigos de formato: Esperado / Encontrado
- Arquivo e data (`lastSeenAt`, fuso `America/Sao_Paulo`)
- Observação sobre múltiplos exames associados ao mesmo código HP

**Código:** `NotificationService.enrichMetadata` → `GoogleChatService.buildHpConn001Message`; `NotificationsModule` importa `MssqlModule`.

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
