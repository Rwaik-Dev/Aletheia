---
tags:
  - site-lsp
  - seguranca
  - auth
projeto: site-lsp
ultima-revisao: '2026-09-18'
---

# Autenticação e segurança — site-lsp

[[Visão Geral]] · [[Painel Admin]] · [[APIs e Integrações]]

## Stack de auth

| Peça | Implementação |
| --- | --- |
| Framework | Auth.js (NextAuth v5 beta) |
| Adapter | `@auth/prisma-adapter` |
| Sessão | Tabela `Session` (database sessions) |
| Senha | Argon2id (`app/lib/auth/password.ts`) |
| MFA | TOTP (`otpauth`) + backup codes; secret criptografado (`MFA_ENCRYPTION_KEY`) |
| Config | `auth.config.ts` + `auth.ts` |
| Handlers | `app/api/auth/[...nextauth]/route.ts` |

Login por **Server Actions** customizadas — não usar JWT para credentials.

## Fluxo de login (detalhado)

1. Validar e-mail/senha; checar `lockedUntil` (lockout).
2. Se MFA habilitado → cookie `mfa_pending` + redirect `/admin/mfa`.
3. Senão → criar `Session` com IP/user-agent opcionais.
4. Falhas incrementam `failedLoginAttempts`; após limite → bloqueio temporário.

## Onboarding completo

Operador só acessa `(panel)` quando:

- `emailVerified` preenchido;
- MFA configurado e verificado (`MfaSecret.enabledAt`).

Fluxo guiado em `/admin/onboarding`.

## Cookies (admin)

| Cookie | Uso |
| --- | --- |
| `authjs.session-token` / `__Secure-authjs.session-token` | Sessão Auth.js |
| `mfa_pending` | Estado entre login e MFA |

Nomes/helpers: `app/lib/auth/cookies.ts`.

## Lockout

- Lógica: `app/lib/auth/account-lockout.ts`.
- Reset de contadores após login bem-sucedido.

## Reset de senha e verificação de e-mail

- Tokens hasheados: `PasswordReset`, `VerificationToken`.
- Links usam `SITE_URL` (ou paths under `/admin/...`).
- Troca de senha **invalida** sessões existentes.

## Auditoria

Modelo `AuditLog`: eventos de login, MFA, alterações sensíveis; IP/user-agent/metadata JSON.
Retenção **365 dias** (ver abaixo).

Eventos registrados via `app/lib/auth/audit.ts`.

## Rate limits

Persistidos em `RateLimitEntry` (`app/lib/rateLimit/`).

| Key (padrão) | Limite | Janela |
| --- | --- | --- |
| `login:{ip}:{email}` | 5 | 10 min |
| `mfa:{ip}:{userId}` | 3 | 10 min |
| `password-reset:{ip}:{email}` | 5 | 10 min |
| `mfa-setup-begin:{ip}:{adminId}` | 3 | 10 min |
| `mfa-setup:{ip}:{adminId}` | 10 | 10 min |
| `email-verification:{ip}:{adminId}` | 5 | 10 min |
| `contact:{ip}:{email}` | 5 | 10 min |
| `analytics:{ip}` | 120 | 1 min |

## IP confiável

- `TRUSTED_IP_HEADER` — **obrigatório em produção** para rate limit e analytics corretos.
- Sem header confiável, clientes caem no bucket `unknown` (`app/lib/request/client-ip.ts`).
- Em dev, `x-forwarded-for` ainda aceito.

## Headers de segurança

`next.config.ts`: Referrer-Policy, X-Content-Type-Options, X-Frame-Options, Permissions-Policy, HSTS (prod).

`proxy.ts`: CSP completa; nonce em prod para scripts dinâmicos.

## Retenção de dados

Arquivo: `app/lib/security/data-retention.ts` — `pruneRetentionData()` oportunista (~1 h).

| Dado | Retenção |
| --- | --- |
| `AnalyticsEvent` | 90 dias |
| `AuditLog` | 365 dias |

## Validação e mascaramento

- Zod em APIs e actions.
- E-mail HTML sanitizado (`app/lib/mail/sanitize.ts`); templates em `templates.ts`.
- Mascaramento de PII em logs (`app/lib/security/mask.ts`).

## Health check

`GET /api/health`: verifica app + DB.
- Produção: sem `HEALTH_CHECK_TOKEN` → **401**; com token → `Authorization: Bearer <token>`.

## Checklist produção

- [ ] `AUTH_SECRET`, `MFA_ENCRYPTION_KEY`, `ANALYTICS_HASH_SALT` fortes e únicos
- [ ] `SITE_URL` HTTPS canônico
- [ ] `TRUSTED_IP_HEADER` alinhado ao proxy
- [ ] `HEALTH_CHECK_TOKEN` para monitoramento
- [ ] Seed admin com senha ≥12 chars; depois rotacionar se necessário
- [ ] MFA habilitado para todos os operadores
