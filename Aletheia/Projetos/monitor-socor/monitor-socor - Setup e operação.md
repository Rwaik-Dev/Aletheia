# monitor-socor — Setup e operação

Cliente **MV (SOCOR)** que lê `*_alerta.txt`, classifica erros e envia snapshot ao **Apex** (`integration: MV`, auth `X-API-Key`).

## Pré-requisitos

| Requisito | Detalhe |
|-----------|--------|
| SO | Windows Server (serviço + DPAPI) |
| Python | 3.10+ (dev) |
| Apex | URL base + API Key |
| Pasta | Caminho dos alertas MV (`PASTA_ERROS`) |

## Dados locais

| Modo | Diretório |
|------|-----------|
| Produção (exe / `MONITOR_DEV_MODE=0`) | `%PROGRAMDATA%\MonitorSocor\` |
| Dev (`python app.py`) | `%LOCALAPPDATA%\MonitorSocorDev\` |

Arquivos típicos: `config.db`, `monitor.log`, `estado_erros.json`, `monitor.lock`.

## Serviço Windows

- **Nome SCM:** `MonitorSocor`
- **Display:** Monitor SOCOR MV-Apex
- **Start:** automatico (`start=auto`)
- Variável de ambiente no serviço: `MONITOR_DATA_DIR` (alinhada à UI)

Instalação: aba **Execução** do `MonitorSocorConfig.exe` ou `python monitor_service.py --startup=auto install` (Admin).

## UI

`python app.py` ou `MonitorSocorConfig.exe` — abas Configuração, Execução, Logs.

## Fluxo do ciclo

1. `*_alerta.txt` em `pasta_erros`
2. Parse + classificação
3. Estado local + expiração por inatividade
4. POST ingest Apex
5. Move para `pasta_erros/analisados/`

## Referências

- Repositório: `C:\Estudos\monitor-socor`
- Padrão operacional: [[Projetos/monitor-hp/Historico de Desenvolvimento|monitor-hp]] / monitor-integracao
- Contrato Apex: [[Projetos/apex/Regras de Negocio|Regras de Negocio — Apex]]
