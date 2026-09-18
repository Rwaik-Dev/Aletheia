# Runbook - Monitor HP

## Configurar

```powershell
cd C:\Estudos\monitor-hp
pip install -r requirements.txt
python scripts/generate_icon.py
python app.py
```

UI no padrão [[Projetos/monitores/UI Desktop - Padrão Apex Clients|Apex Clients]] (tema **Exibir**, footer copia caminho do `config.db`).

Deploy GUI + serviço (mesma pasta):

```powershell
pyinstaller MonitorHPConfig.spec
pyinstaller MonitorHPService.spec
```

→ `dist\MonitorHPConfig.exe` + `dist\MonitorHPService.exe`

Preencher:

- Pasta erros
- Pasta analisados
- Pasta relatorios
- URL base Apex
- Client ID e Client Secret (service client Apex)
- Source `10.2.30.53`
- Integration `HP`

## Executar manualmente

```powershell
python processador.py
```

## Servico Windows

Executar como Administrador:

```powershell
python monitor_service.py --startup=auto install
python monitor_service.py start
```

## Testar

```powershell
$env:MONITOR_TEST_CRYPTO='1'
$env:MONITOR_SKIP_ACL='1'
python -m pytest -q
```

## Troubleshooting

- **Acesso negado ao iniciar servico (WinError 5):** gravar `MONITOR_DATA_DIR` no registro exige Administrador. Feche o app, clique direito em `python app.py` ou no exe → Executar como administrador → **Instalar servico** ou **Iniciar** uma vez. Depois o servico pode usar o caminho ja gravado.
- Sem envio: verificar `CLIENT_ID` e `CLIENT_SECRET` na UI.
- Sem leitura: verificar se existem `.htm` ou `.html` em `PASTA_ERROS`.
- Arquivo nao movido: verificar permissao em `PASTA_AVALIADOS`.
- Encoding estranho: parser tenta `utf-8`, `latin-1` e `cp1252`.
