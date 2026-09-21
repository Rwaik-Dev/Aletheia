---
tags:
  - site-lsp
  - admin
  - cms
projeto: site-lsp
ultima-revisao: '2026-09-21'
---

# Painel admin — site-lsp

[[Visão Geral]] · [[Autenticação e Segurança]] · [[Modelo de Dados]]

Fonte de rotas: `app/lib/routes.ts` (`ROUTES`).

## Rotas públicas (sem sessão completa)

| Rota | Propósito |
| --- | --- |
| `/admin/login` | Login e-mail/senha |
| `/admin/mfa` | Challenge TOTP / backup codes |
| `/admin/onboarding` | E-mail não verificado ou MFA pendente |
| `/admin/forgot-password` | Solicitar reset |
| `/admin/reset-password` | Token de reset |
| `/admin/verify-email` | Confirmação de e-mail |

## Painel autenticado `(panel)`

Layout: `app/admin/(panel)/layout.tsx` — sidebar (`AdminSidebar`), topbar, tema documento.

| Rota | Módulo | Entidade Prisma |
| --- | --- | --- |
| `/admin` | Dashboard | resumo |
| `/admin/analytics` | Analytics | `AnalyticsEvent` agregado |
| `/admin/banners` | Banners | `Banner` |
| `/admin/servicos` | Serviços | `Service` |
| `/admin/unidades` | Unidades | `Unit` |
| `/admin/preparo-exames` | Preparo | `ExamPrep` |
| `/admin/sobre-nos` | Sobre | `AboutSection` (singleton `id=default`) |
| `/admin/calendarios-vacinais` | Vacinas | `VaccineCalendar` + `VaccineCalendarItem` |
| `/admin/convenios` | Convênios | `HealthInsurance` |
| `/admin/usuarios` | Usuários | `User` (role `ADMIN`) |
| `/admin/settings/security` | Segurança | MFA setup, troca de senha |

Padrão CRUD: listagem → `new` → `[id]/edit`.

## Uploads por módulo

| Módulo | Diretório | `storage.ts` |
| --- | --- | --- |
| Banners | `public/banners/` | sim |
| Unidades | `public/units/` | sim |
| Calendários vacinais | `public/vacinas/` | sim |
| Convênios | `public/convenios/` | sim |
| Serviços / preparo / sobre | — | ícones ou só texto |

Processamento de imagem com **sharp**; nomes de arquivo sanitizados.

## Server Actions principais

| Área | Arquivo |
| --- | --- |
| Login / logout | `app/admin/actions.ts` |
| Onboarding | `app/admin/onboarding/actions.ts` |
| MFA challenge | `app/admin/mfa/actions.ts` |
| Security settings | `app/admin/settings/actions.ts` |
| CRUD por entidade | `app/admin/<modulo>/actions.ts` |

Helpers compartilhados: `app/lib/admin/action-utils.ts` (mensagens de erro, revalidação).

Após mutação de conteúdo público, actions disparam revalidação da home (paths em `ADMIN_REVALIDATE_PATHS`).

## Analytics no admin

- Ingestão continua no **site** (`POST /api/analytics`).
- Dashboard admin lê `AnalyticsEvent` com filtros por período, path, device, etc. (`app/admin/(panel)/analytics/page.tsx` + lib de agregação).

## UX / acessibilidade admin

- Formulários: **react-hook-form** + **Zod** resolvers.
- Tabelas com ações editar/excluir/reordenar (`displayOrder`).
- Flag `isActive` para ocultar conteúdo no site sem apagar registro.

## Deploy

O painel vive em `app/admin/` no **mesmo** repositório e deploy que o site público (`/` e `/admin` no mesmo host). Após alterações de conteúdo, Server Actions invalidam a home in-process. Em produção: MFA, senhas fortes e, se possível, restrição de `/admin` no reverse proxy — ver [[Modelo de Deploy]].
