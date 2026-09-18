---
tags:
  - site-lsp
  - testes
  - vitest
  - playwright
projeto: site-lsp
ultima-revisao: '2026-09-18'
---

# Testes — site-lsp

[[Runbook]] · [[Visão Geral]]

## Ferramentas

| Tipo | Stack | Comando |
| --- | --- | --- |
| Unitário / integrado | Vitest + Testing Library + jsdom | `npm run test` |
| Cobertura | v8 | `npm run test:coverage` |
| E2E | Playwright | `npm run test:e2e` |
| Watch | Vitest | `npm run test:watch` |

Config: Vitest via projeto Next/TS; Playwright em `tests/e2e/`.

## Unitários (`tests/unit/`)

Cobertura de fluxos críticos:

| Arquivo (exemplos) | Área |
| --- | --- |
| `auth-session.test.ts`, `auth-onboarding.test.ts` | Sessão e onboarding |
| `mfa-actions.test.ts`, `admin-guards.test.ts` | MFA e guards |
| `password.test.ts`, `users-actions.test.ts` | Senha e usuários |
| `contact-route.test.ts`, `analytics-lib.test.ts` | APIs públicas |
| `health-route.test.ts`, `client-ip.test.ts` | Health e IP |
| `public-content.test.ts` | Cache/fallback home |
| `admin-actions.test.ts`, `about-actions.test.ts` | Server Actions |
| `mail-templates.test.ts` | E-mail |
| `hero-section.test.tsx`, `vaccine-calendar-form.test.tsx` | Componentes |

## E2E (`tests/e2e/`)

- `services.spec.ts` — fluxos como login admin, CRUD serviço, contato.
- Sobe app com `DISABLE_EMAIL_DELIVERY=true`.
- Requer `DATABASE_URL` acessível e migrações aplicadas.
- `PLAYWRIGHT_BASE_URL` opcional se app já estiver rodando.

## Boas práticas no repo

- Dados E2E isolados; não depender do seed manual para IDs fixos frágeis.
- `ENABLE_TEST_LOGS=true` reativa logs durante testes se necessário.

## CI (referência)

Rodar antes de release:

```bash
npm run lint
npm run test
npm run build
# npm run test:e2e  # com DB de CI
```
