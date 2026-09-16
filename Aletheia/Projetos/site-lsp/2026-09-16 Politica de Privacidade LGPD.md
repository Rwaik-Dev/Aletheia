---
tags:
  - site-lsp
  - lgpd
  - privacidade
  - site-publico
status: implementado
data: '2026-09-16T00:00:00.000Z'
projeto: site-lsp
repositorio: 'C:/Estudos/site-lsp'
---

# Política de Privacidade (LGPD) — site público

## Contexto

Foi avaliada a necessidade do **banner convencional de cookies** no site institucional. Conclusão técnica:

- Na **área pública**, o visitante **não recebe cookies** definidos pela aplicação.
- Cookies existem apenas em **`/admin`** (sessão Auth.js e `mfa_pending`) — estritamente necessários ao painel interno.
- Métricas de uso usam **analytics próprio** com identificadores em `localStorage` / `sessionStorage` (`lsp_analytics_visitor`, `lsp_analytics_session`), enviados para `POST /api/analytics`.
- No servidor: hashes de visitante/sessão/IP, user-agent, referrer; retenção **90 dias** (`app/lib/security/data-retention.ts`).
- **Sem** Google Analytics, Meta Pixel ou ferramentas equivalentes de marketing.

Portanto, o banner genérico “este site usa cookies” **não é o instrumento adequado** para o site público. O que a LGPD pede com mais propriedade é **transparência** sobre tratamento de dados (formulário de contato + métricas + direitos do titular).

## Decisão

1. **Não** implementar banner de consentimento de cookies na home.
2. Publicar **Política de Privacidade** dedicada em `/privacidade`, alinhada ao comportamento real do código.
3. Linkar a política no **rodapé** e aviso curto no **formulário Fale Conosco**.

## Implementação (2026-09-16)

| Artefato | Descrição |
| --- | --- |
| `app/privacidade/page.tsx` | Página institucional com seções: controlador, dados tratados, finalidades/bases, contato, analytics (storage local), cookies só no admin, retenção, segurança, direitos LGPD, alterações. Metadata SEO. Inclui `NavBar`, `Footer`, `SiteAnalyticsTracker`. |
| `app/components/Footer.tsx` | Link **Política de Privacidade** em Links Úteis (`/privacidade`); evento analytics `footer_privacy`. |
| `app/components/ContactForm.tsx` | Texto antes do envio: concordância com tratamento conforme política (link `/privacidade`). |

**URL pública:** `{SITE_URL}/privacidade`

**Última atualização declarada na página:** 16 de setembro de 2026.

## O que a política descreve (espelho do código)

### Formulário de contato

- Campos: nome, e-mail, assunto, mensagem.
- Fluxo: validação Zod → rate limit → envio via `sendMail` (`app/api/contact/route.ts`); **não** persiste mensagem em tabela pública do site.
- Honeypot (`website`) e limite por IP/e-mail.

### Analytics

- Cliente: `app/lib/analytics-client.ts` + `SiteAnalyticsTracker`.
- Servidor: `app/lib/analytics.ts` → `AnalyticsEvent` no PostgreSQL.
- Pseudonimização por hash com `ANALYTICS_HASH_SALT`.

### Cookies (transparência)

- Visitante do site público: **nenhum cookie** da app.
- Admin: `authjs.session-token` / `__Secure-authjs.session-token`, `mfa_pending` (documentado em `DOCUMENTATION.md` §12.4).

## Pendências / melhorias opcionais

- Incluir **CNPJ**, endereço ou **e-mail dedicado** de privacidade/DPO na seção do controlador (hoje orienta Fale Conosco + canais do rodapé).
- Revisão jurídica interna/compliance do texto antes de produção.
- Atualizar `DOCUMENTATION.md` na raiz do repo se quiser manter paridade com a wiki Obsidian.
- Após [[ADR-001 Separacao do painel CMS em site-lsp-admin|ADR-001]]: política continua válida no site público; admin permanece com cookies de sessão fora do escopo do visitante.

## Relacionado

- Repositório: `C:\Estudos\site-lsp`
- Documentação técnica: `DOCUMENTATION.md` (analytics, cookies admin, retenção)
- [[ADR-001 Separacao do painel CMS em site-lsp-admin]]

## Referência rápida — arquivos tocados

```text
app/privacidade/page.tsx      (novo)
app/components/Footer.tsx     (link)
app/components/ContactForm.tsx (aviso LGPD)
```
