---
tags:
  - site-lsp
  - api
projeto: site-lsp
ultima-revisao: '2026-09-18'
---

# APIs e integrações — site-lsp

[[Site Público]] · [[Autenticação e Segurança]] · [[Runbook]]

## Route Handlers

### `POST /api/contact`

- **Arquivo:** `app/api/contact/route.ts`
- **Body:** nome, e-mail, assunto, mensagem (+ honeypot `website` deve ficar vazio)
- **Validação:** `app/lib/contactSchema.ts` (Zod)
- **Proteção:** rate limit `contact:{ip}:{email}`
- **Efeito:** envia e-mail via `app/lib/mail/index.ts` para `MAIL_TO` (opcional BCC)
- **Persistência:** nenhuma tabela de mensagens no site

### `POST /api/analytics`

- **Arquivo:** `app/api/analytics/route.ts`
- **Payload:** eventType (`PAGEVIEW` | `EVENT`), eventName, path, visitor/session hashes, metadata opcional
- **Proteção:** rate limit `analytics:{ip}` (120/min)
- **Persistência:** `AnalyticsEvent` — IP/visitor pseudonimizados com `ANALYTICS_HASH_SALT`
- **Cliente:** `app/lib/analytics-client.ts`

### `GET /api/health`

- **Arquivo:** `app/api/health/route.ts`
- **Resposta:** status app + ping PostgreSQL
- **Auth:** Bearer `HEALTH_CHECK_TOKEN` quando configurado (obrigatório em produção para acesso)

### `GET|POST /api/auth/[...nextauth]`

- **Arquivo:** `app/api/auth/[...nextauth]/route.ts`
- **Export:** handlers de `auth.ts`
- Login principal via Server Actions; handlers para compatibilidade Auth.js / OAuth futuro

## E-mail (Gmail SMTP)

| Variável | Papel |
| --- | --- |
| `MAIL_USER` | Conta Gmail |
| `GOOGLE_APP_PASSWORD` | App password |
| `MAIL_FROM` | Remetente (default = user) |
| `MAIL_TO` | Destinatário contato |
| `MAIL_BCC` | Cópia oculta opcional |
| `DISABLE_EMAIL_DELIVERY` | `true` → só log, não envia |

Templates: reset de senha, verificação de e-mail (`app/lib/mail/templates.ts`).

## Integrações externas

| Serviço | Uso |
| --- | --- |
| PostgreSQL | Dados + sessões + rate limits + analytics |
| Gmail | Contato + transacionais admin |
| WhatsApp (link) | CTA fixo na home — URL hardcoded no componente |
| Portal resultados | Link no footer — externo |

## Endpoint futuro (ADR-001)

`POST /api/revalidate` no site público — Bearer `REVALIDATE_SECRET` — para invalidar cache quando o CMS estiver em outro repositório. **Ainda não implementado.**

## Observabilidade

- Logs estruturados: `app/lib/logger.ts` (server-side).
- Testes: `tests/unit/contact-route.test.ts`, `health-route.test.ts`, `analytics-lib.test.ts`.
