# monitor-socor — Setup e operação

Cliente **MV (SOCOR)** que lê `*_alerta.txt`, classifica erros e envia snapshot ao **Apex** (`integration: MV`).

Auth: **service client** → `POST /api/v1/auth/token`, ingest com `Authorization: Bearer` e `resolveAbsent: true`.

→ Detalhes: [[Projetos/monitor-socor/Visao Geral|Visão geral]] · [[Projetos/monitor-socor/Runbook|Runbook]] · [[Projetos/monitor-socor/Regras de Negocio|Regras de negócio]] · [[Projetos/apex/API - Guia Novos Clients|Guia Apex]]

## Pré-requisitos

| Requisito | Detalhe |
|-----------|--------|
| SO | Windows (serviço, DPAPI, pywin32) |
| Python | 3.10+ (desenvolvimento) |
| Apex | URL base (`:3333`) + service client (`CLIENT_ID` / `CLIENT_SECRET`) |
| Pasta MV | Caminho dos alertas (`pasta_erros`; absoluto no serviço) |

## Configuração segura

Substitui `.env` em produção:

| Item | Local |
|------|--------|
| Parâmetros públicos | SQLite `config.db` |
| `CLIENT_ID` / `CLIENT_SECRET` | Mesmo banco, cifrados com **DPAPI** (máquina) |

| Modo | Diretório |
|------|-----------|
| Produção (exe / `MONITOR_DEV_MODE=0`) | `%PROGRAMDATA%\MonitorSocor\` |
| Dev (`python app.py`) | `%LOCALAPPDATA%\MonitorSocorDev\` |

Arquivos típicos: `config.db`, `monitor.log`, `estado_erros.json`, `monitor.lock`.

## UI (`app.py` / `MonitorSocorConfig.exe`)

Abas:

1. **Configuração** — pastas, Apex, intervalos; Salvar; Testar conexão; Importar `.env`
2. **Execução** — status / instalar / iniciar / parar / reiniciar serviço (auto-start)
3. **Logs** — tail de `monitor.log`

## Serviço Windows

- **Nome SCM:** `MonitorSocor`
- **Display:** Monitor SOCOR MV-Apex
- **Start:** automático
- **Dados:** `MONITOR_DATA_DIR` alinhado à UI (registro do serviço)

Instalação: aba **Execução** ou `python monitor_service.py --startup=auto install` (Admin).

Reinicie o serviço após salvar mudanças na UI.

## Fluxo do ciclo

1. Lista `*_alerta.txt` em `pasta_erros`
2. Parse até `**** Fim`; classificação por descrição
3. Atualiza `estado_erros.json`; expira inativos (`intervalo_inatividade_minutos`)
4. Token JWT + `POST .../errors/ingest` com erros ativos
5. Move arquivos para `pasta_erros/analisados/`

## Execução rápida

```powershell
pip install -r requirements.txt
python app.py                    # configurar
python processador.py            # loop manual
python monitor_service.py start  # serviço (após install)
```

## Referências

- Repositório: `C:\Estudos\monitor-socor`
- README no repo (espelho operacional)
- Irmãos: [[Projetos/monitor-hp/Visao Geral|monitor-hp]], monitor-integracao
- Apex: [[Projetos/apex/Regras de Negocio|Regras de Negócio — Apex]]
