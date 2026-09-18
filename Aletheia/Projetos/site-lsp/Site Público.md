---
tags:
  - site-lsp
  - site-publico
projeto: site-lsp
ultima-revisao: '2026-09-18'
---

# Site público — site-lsp

[[Visão Geral]] · [[APIs e Integrações]] · [[2026-09-16 Politica de Privacidade LGPD]]

## Páginas

| Rota | Arquivo | Notas |
| --- | --- | --- |
| `/` | `app/page.tsx` | Home institucional |
| `/privacidade` | `app/privacidade/page.tsx` | Política LGPD; link no footer e aviso no contato |

## Ordem das seções na home

1. Hero / banners (`HeroSection`)
2. Diferenciais
3. Serviços
4. Calendários vacinais
5. Preparo de exames
6. Sobre nós
7. Unidades
8. Soluções corporativas (conteúdo fixo/institucional)
9. Convênios (`HealthInsurances`)
10. Contato (`ContactForm`)

Componentes principais em `app/components/`.

## Origem do conteúdo

- **Dinâmico:** PostgreSQL via `app/lib/public-content.ts` (entidades ativas, ordenadas por `displayOrder`).
- **Cache:** 5 min + invalidação quando o admin altera dados.
- **Fallback:** listas estáticas se DB vazio ou erro de conexão.

## Links externos fixos (não vêm do CMS)

- WhatsApp de agendamento de coleta (CTA fixo na hero).
- Redes sociais e contatos no `Footer`.
- Resultados de exames (portal externo, conforme componentes/footer).

## Formulário de contato

- UI: `ContactForm.tsx` — aviso de concordância com [[2026-09-16 Politica de Privacidade LGPD]].
- API: `POST /api/contact` — Zod (`contactSchema`), honeypot `website`, rate limit, e-mail via `app/lib/mail`.
- **Não persiste** mensagem em tabela pública; envia e-mail para `MAIL_TO`.

## Analytics (visitante)

- **Sem** Google Analytics / Meta Pixel.
- Cliente: `app/lib/analytics-client.ts` + tracker na layout/páginas.
- Identificadores em `localStorage` / `sessionStorage`: `lsp_analytics_visitor`, `lsp_analytics_session`.
- Servidor: `POST /api/analytics` → modelo `AnalyticsEvent` (hashes com `ANALYTICS_HASH_SALT`).
- Eventos de UI (ex.: `footer_privacy`) via mesmo pipeline.

### Cookies no site público

O visitante da home **não recebe cookies** definidos pela aplicação. Cookies Auth.js e `mfa_pending` existem **apenas** em `/admin`. Ver nota LGPD.

## Ícones de serviço

O admin restringe ícones a um conjunto permitido (Heroicons names) — validado no schema do módulo serviços; a home renderiza conforme o valor salvo.

## SEO / metadata

- Layout raiz e páginas definem metadata Next.js; `SITE_URL` alimenta Open Graph em produção (HTTPS obrigatório via `getSiteUrl()`).

## Arquivos estáticos de mídia

| Pasta | Uso |
| --- | --- |
| `public/banners/` | Imagens de hero |
| `public/units/` | Fotos de unidades |
| `public/vacinas/` | Imagens de calendários vacinais |
| `public/convenios/` | Logos de convênios |

Em Docker, essas pastas são volumes nomeados (persistem entre recreates).
