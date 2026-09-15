# API Apex — Guia para novos clients

Voltar: [[Projetos/apex/API - Visao Geral|Visão geral]]

## Escolha do tipo de client

| Cenário | Autenticação | Endpoints típicos |
|---------|--------------|-------------------|
| Daemon / script de monitoramento | Service client | `POST .../errors/ingest` |
| Ferramenta CLI interna | Service client ou user | erros + signum |
| Aplicação web (browser) | User login + refresh | erros, signum; admin se `ADMIN` |
| CI / health check | Nenhum | `GET /health/ready` |

**Nunca** embutir `clientSecret` em frontend público. Service credentials só em servidor ou vault local.

---

## Fluxo A — Service client (ingest)

1. Admin cria client: `POST /api/v1/admin/service-clients` → guardar `clientSecret`
2. Obter token:

```http
POST /api/v1/auth/token
Content-Type: application/json

{"clientId":"meu-client","clientSecret":"..."}
```

3. Enviar snapshot completo periodicamente:

```http
POST /api/v1/errors/ingest
Authorization: Bearer <accessToken>
Content-Type: application/json
```

4. Renovar token antes de `expiresIn` (sem refresh para service).

### Python (referência)

```python
import time
import requests

BASE = "http://localhost:3333"
CLIENT_ID = "meu-client"
CLIENT_SECRET = "..."

cache: dict = {}

def access_token() -> str:
    if cache.get("t") and cache.get("exp", 0) > time.time():
        return cache["t"]
    r = requests.post(
        f"{BASE}/api/v1/auth/token",
        json={"clientId": CLIENT_ID, "clientSecret": CLIENT_SECRET},
        timeout=30,
    )
    r.raise_for_status()
    data = r.json()
    cache["t"] = data["accessToken"]
    cache["exp"] = time.time() + data["expiresIn"] - 30
    return cache["t"]

def ingest(integration: str, source: str, errors: list[dict], resolve_absent: bool = True):
    r = requests.post(
        f"{BASE}/api/v1/errors/ingest",
        json={
            "integration": integration,
            "source": source,
            "resolveAbsent": resolve_absent,
            "errors": errors,
        },
        headers={"Authorization": f"Bearer {access_token()}"},
        timeout=60,
    )
    r.raise_for_status()
    return r.json()
```

### Boas práticas ingest

- Enviar **todos** os erros ativos da combinação `integration` + `source` em cada ciclo.
- Usar `resolveAbsent: false` se o client não controla snapshot completo (ex.: só append de novos eventos).
- Estabilizar `message`, `errorCode`, `filePath` para deduplicação correta.
- Para notificação Tasy Google Chat, usar `errorCode` exatos: `mapeamento` ou `tabela de preço`.

---

## Fluxo B — Frontend (usuário humano)

1. `POST /api/v1/auth/login` → guardar `accessToken` + `refreshToken` (memória segura / httpOnly se BFF)
2. Chamadas API com `Authorization: Bearer <accessToken>`
3. Ao receber **401**, `POST /api/v1/auth/refresh` com refresh token → substituir ambos tokens
4. Logout: `POST /api/v1/auth/logout` + apagar tokens locais

Base URL da API no Next: `NEXT_PUBLIC_API_URL=http://localhost:3333` (exemplo).

Detalhes de telas: `docs/frontend-integration.md` no repo.

---

## Fluxo C — Consultas Signum

Mesmo Bearer do fluxo A ou B.

```http
GET /api/v1/status/fila/tasy
Authorization: Bearer <accessToken>
```

Tratar **sempre** campo `error` boolean no JSON antes de usar `data`.

---

## Fluxo D — Provisionamento (ADMIN)

1. Login admin
2. Criar users / service clients conforme [[Projetos/apex/API - Catalogo de Endpoints#Admin — service clients|catálogo admin]]
3. Rotacionar secret se vazamento: `POST .../rotate-secret`

---

## Ambientes

| | Local | Produção |
|--|-------|----------|
| API | `:3333` | `:3333` (10.2.30.102) |
| Postgres | `:5433` host | `:5432` no host |
| Frontend | `:3000` | conforme deploy |

---

## Testes de fumaça (curl)

```bash
# Health
curl -s http://localhost:3333/health/ready

# Token
curl -s -X POST http://localhost:3333/api/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{"clientId":"apex-ingest-client","clientSecret":"<secret>"}'

# List errors
curl -s "http://localhost:3333/api/v1/errors?limit=10" \
  -H "Authorization: Bearer <jwt>"
```

---

## Referências cruzadas

- [[Projetos/apex/API - Catalogo de Endpoints|Catálogo completo]]
- [[Projetos/apex/API - Autenticacao|Autenticação]]
- [[Projetos/apex/Regras de Negocio|Regras de negócio]]
- [[Projetos/monitor-hp/Regras de Negocio|Monitor HP]] — exemplo de client externo (validar enum `integration` antes de produção)
