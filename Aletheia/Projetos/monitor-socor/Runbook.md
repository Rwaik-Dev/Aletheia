# Runbook — Monitor SOCOR

## Desenvolvimento — configurar

```powershell
cd C:\Estudos\monitor-socor
pip install -r requirements.txt
python app.py
```

Na aba **Configuração**, preencher:

- **Pasta de erros** — diretório onde a MV grava `*_alerta.txt` (preferir caminho absoluto)
- **URL base Apex** — ex.: `https://servidor:3333` (sem `/api/v1/errors/ingest`)
- **Client ID** e **Client Secret** — service client Apex
- **Source** — identificador do servidor MV (default: hostname)
- **Integration** — `MV`
- Intervalos de ciclo e inatividade

Ações úteis: **Salvar**, **Testar conexão**, **Importar .env**.

Secrets na UI: campo vazio = manter valor atual; novo valor = rotacionar; **Limpar** = remove no próximo salvar.

### Modo dev vs produção (caminho do banco)

```powershell
$env:MONITOR_DEV_MODE = "1"   # %LOCALAPPDATA%\MonitorSocorDev\
$env:MONITOR_DEV_MODE = "0"   # %PROGRAMDATA%\MonitorSocor\ + ACL
```

## Migração do .env

Na primeira abertura sem SQLite, ou via **Importar .env**:

1. Campos conhecidos vão para `config.db` (secrets no DPAPI)
2. Arquivo renomeado para `.env.migrated`

Mapeamento principal: `INTERVALO_SEGUNDOS` ou `INTERVALO_PROCESSADOR_SEGUNDOS` → `intervalo_processador_segundos`.

Campos do `.env`: ver `.env.example` no repositório (somente referência para importação na UI).

## Executar manualmente (homologação)

```powershell
python processador.py
```

Requer config já salva no SQLite (via `app.py`).

## Serviço Windows

Como **Administrador**:

```powershell
python monitor_service.py --startup=auto install
python monitor_service.py start
```

Ou use a aba **Execução** do `app.py` / `MonitorSocorConfig.exe`.

| Campo SCM | Valor |
|-----------|--------|
| Nome | `MonitorSocor` |
| Display | Monitor SOCOR MV-Apex |

Após alterar config na UI, **reinicie o serviço** para recarregar o SQLite.

Debug sem SCM:

```powershell
python monitor_service.py debug
```

## Testes

```powershell
$env:MONITOR_TEST_CRYPTO = "1"
$env:MONITOR_SKIP_ACL = "1"
python -m pytest -q
```

(24 testes na suite atual.)

## Build executáveis

Regenerar ícones (opcional, após editar `assets/icon.svg`):

```powershell
python scripts/generate_icon.py
```

```powershell
pyinstaller MonitorSocorConfig.spec
pyinstaller MonitorSocorService.spec
```

O `MonitorSocorConfig.spec` embute `sv-ttk`, `assets/icon.ico` (ícone do exe) e `icon.png` (janela).

Instalar o serviço a partir de `dist/MonitorSocorService.exe` (Admin).

## Troubleshooting

- **Erro 1063 ao abrir `MonitorSocorService.exe`:** normal se você deu duplo clique ou rodou o exe “para iniciar”. Esse binário só entra no modo serviço quando o **Windows SCM** o chama (após `install` + **Iniciar** na aba Execução ou `services.msc`). Para teste manual use `MonitorSocorService.exe debug`. Instalação: `MonitorSocorService.exe install` (Admin) ou aba Execução do Config.
- **Configuração ausente no serviço:** abrir UI, salvar config, reiniciar serviço. Mensagem: *Configure pelo aplicativo gráfico*.
- **Acesso negado (WinError 5) ao instalar/iniciar serviço ou gravar `MONITOR_DATA_DIR`:** executar UI ou install como Administrador uma vez.
- **Envio ignorado:** verificar `CLIENT_ID` e `CLIENT_SECRET` na UI (não vazios após salvar).
- **Testar conexão falha:** URL base correta, service client ativo no Apex, firewall até a porta do Apex.
- **Nenhum arquivo pendente:** confirmar sufixo `*_alerta.txt` e caminho de `pasta_erros` (absoluto no serviço).
- **Outra instância em execução:** lock em `monitor.lock`; encerrar processo duplicado.
- **Arquivo não movido para analisados:** permissão de escrita em `{pasta_erros}/analisados/`.
- **Encoding:** parser tenta `cp1252` e `latin-1`.

## Referências

- [[Projetos/monitor-socor/Visao Geral|Visão geral]]
- [[Projetos/monitor-socor/Regras de Negocio|Regras de negócio]]
- Padrão operacional: [[Projetos/monitor-hp/Runbook|monitor-hp]]
