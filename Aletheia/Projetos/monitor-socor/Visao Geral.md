# Monitor SOCOR — Visão geral

Cliente Python para **Windows** que monitora arquivos `*_alerta.txt` gerados pela integração **MV (SOCOR)**, classifica os erros de negócio, mantém estado local e envia um **snapshot dos erros ativos** ao backend **Apex** (`integration: MV`).

Repositório: `C:\Estudos\monitor-socor`

## Documentação neste vault

- [[Projetos/monitores/UI Desktop - Padrão Apex Clients|UI Desktop — padrão Apex Clients]] (referência para HP/Tasy)
- [[Projetos/monitor-socor/monitor-socor - Setup e operação|Setup e operação]] — resumo operacional
- [[Projetos/monitor-socor/Runbook|Runbook]] — comandos, deploy e troubleshooting
- [[Projetos/monitor-socor/Regras de Negocio|Regras de negócio]] — parse, classificação e payload Apex
- [[Projetos/monitor-socor/Historico de Desenvolvimento|Histórico de desenvolvimento]]
- Contrato Apex: [[Projetos/apex/API - Guia Novos Clients|Guia Novos Clients]]

## Estrutura do código

| Módulo | Função |
|--------|--------|
| `monitor/` | Parser MV, classificador, estado JSON, cliente Apex (ingest + teste de conexão na UI) |
| `monitor_core.py` | Ciclo do processador, lock de instância única, logging |
| `config.py` | Modelo tipado `Config` e validação |
| `config_repository.py` | SQLite (`config.db`) + secrets cifrados (DPAPI) |
| `secure_storage.py` | Cifragem Windows DPAPI (escopo máquina em produção) |
| `env_migration.py` | Migração única `.env` → SQLite |
| `app.py` | Controller da UI desktop |
| `ui/` | Shell, abas, tema **sv-ttk** (Sun Valley), branding slate/teal |
| `assets/` | `icon.svg`, `icon.png`, `icon.ico` (janela + exe) |
| `processador.py` | Loop manual (SQLite) |
| `monitor_service.py` | Serviço Windows + loop |
| `service_control.py` | Instalar/iniciar/parar serviço **MonitorSocor** |
| `tests/` | Config, Apex, migração, serviço, smoke de tema |

## Interface (UI)

Detalhes completos: [[Projetos/monitores/UI Desktop - Padrão Apex Clients|UI Desktop — padrão Apex Clients]].

- Tema **Sun Valley** via `sv-ttk`; menu **Exibir** → tema claro/escuro.
- Cabeçalho com ícone, chip Dev/Produção; rodapé com caminho do `config.db` (clique copia).
- Aba Configuração scrollável, **Procurar…** para pasta de erros, banners de feedback.
- Aba Execução com card de status do serviço; Logs monoespaçados com auto-atualização opcional.

## Modos de execução

| Modo | Quando usar |
|------|-------------|
| Serviço Windows **MonitorSocor** | Produção (auto-start, roda sem UI) |
| `python app.py` / `MonitorSocorConfig.exe` | Configurar secrets e controlar serviço |
| `python processador.py` | Homologação / loop manual com SQLite |

O serviço **não** lê `.env`; configuração vem do SQLite (migração única via UI).

## Dados locais

| Modo | Diretório |
|------|-----------|
| Produção (exe ou `MONITOR_DEV_MODE=0`) | `%PROGRAMDATA%\MonitorSocor\` |
| Desenvolvimento (`python app.py`, não frozen) | `%LOCALAPPDATA%\MonitorSocorDev\` |
| Override | variável `MONITOR_DATA_DIR` (gravada no registro ao instalar serviço) |

Arquivos típicos: `config.db`, `monitor.log`, `estado_erros.json`, `monitor.lock`.

Em produção, a pasta `ProgramData\MonitorSocor` recebe ACL restrita (Administradores + SYSTEM).

## Defaults MV

- `pasta_erros`: `erros` (relativo ao diretório de trabalho do processo)
- `apex_integration`: `MV`
- `apex_source`: hostname da máquina (se vazio na config)
- `intervalo_processador_segundos`: 180
- `intervalo_inatividade_minutos`: 30
- Analisados: `{pasta_erros}/analisados/`

No serviço, o CWD é `MONITOR_DATA_DIR`; configure `pasta_erros` com caminho absoluto da pasta de alertas MV.

## Autenticação Apex

**Service client** (não API key estática):

1. `POST {APEX_URL}/api/v1/auth/token` com `clientId` / `clientSecret`
2. `POST {APEX_URL}/api/v1/errors/ingest` com `Authorization: Bearer` e `resolveAbsent: true`

Provisionamento: `POST /api/v1/admin/service-clients` no Apex.

## Deploy (PyInstaller)

- `MonitorSocorConfig.spec` → `dist/MonitorSocorConfig.exe` (UI)
- `MonitorSocorService.spec` → `dist/MonitorSocorService.exe` (serviço)

O banco e secrets **não** são embutidos. Backup de `config.db` só vale na **mesma máquina** (DPAPI).

## LGPD

Logs e `estado_erros.json` podem conter identificadores de pacientes. Restringir acesso no servidor MV.
