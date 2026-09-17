# UI Desktop — Padrão Apex Clients (Windows)

Guia transversal para aplicativos **Tkinter** de configuração e controle de **monitores de integração** (MV/SOCOR, HP, Tasy, etc.) que enviam erros ao [[Projetos/apex/Visao Geral|Apex]].

**Referência implementada:** repositório `monitor-socor` (`C:\Estudos\monitor-socor`) — pacote `ui/`, `app.py` como controller.

## Escopo

| Incluído | Fora do escopo |
|----------|----------------|
| Visual, layout, UX comum das abas Config / Execução / Logs | Regras de parse/negócio por integração |
| Tema, tipografia, ícone, PyInstaller Config | UI web (ex.: [[Projetos/xml-reader/xml-reader - Documentação técnica|xml-reader]] React) |
| Padrão serviço Windows + sem flash de console | Lógica de ingest Apex (ver [[Projetos/apex/API - Guia Novos Clients|Guia Novos Clients]]) |

## Stack

| Camada | Escolha |
|--------|---------|
| Toolkit | **Tkinter** + **ttk** |
| Tema | **[Sun Valley](https://github.com/rdbende/Sun-Valley-ttk-theme)** via pacote **`sv-ttk`** |
| SO alvo | Windows 10/11 (serviço + DPAPI) |
| Empacotamento | **PyInstaller** — dois exe: `*Config` (GUI) + `*Service` (SCM) |
| Lint | **Ruff** (`pyproject.toml`, `python -m ruff check .`) |

Menu **Exibir → Tema claro / escuro / alternar** (opcional mas recomendado).

## Identidade visual (neutra)

Paleta **slate + teal** — definida em `ui/branding.py` (copiar/adaptar por client).

| Token | Hex | Uso |
|-------|-----|-----|
| Texto principal | `#0f172a` | Labels |
| Texto secundário | `#64748b` | Hints, footer, `Muted.TLabel` |
| Acento | `#0d9488` | Chip ambiente, links visuais |
| Sucesso | `#059669` | Status serviço OK, banner |
| Aviso | `#d97706` | Pending, alertas |
| Erro | `#dc2626` | Falhas, não instalado |
| Fundo log | `#f8fafc` / texto `#1e293b` | Aba Logs |
| Banners | `#ecfdf5` (ok), `#fef2f2` (erro), `#f1f5f9` (neutro) | Feedback inline |

**Tipografia (Windows):**

- Corpo / formulário: **Segoe UI 10**
- Seções: **Segoe UI 11 bold**
- Título janela (header): **Segoe UI 16 bold**
- Subtítulo header: **Segoe UI 10**
- Logs: **Consolas 9**

**Ícone:** `assets/icon.svg` → `python scripts/generate_icon.py` → `icon.png` (janela) + `icon.ico` (exe). Cores do ícone podem variar por produto; manter legibilidade em 16×16.

## Arquitetura de módulos (`ui/`)

```
ui/
  branding.py    # APP_TITLE, APP_SUBTITLE, cores, fontes, iconphoto (dev + _MEIPASS)
  theme.py       # sv_ttk.set_theme, estilos Primary / Muted / Status* / Accent
  shell.py       # Header + Notebook + footer (caminho config.db, clique = copiar)
  widgets.py     # section_header, scroll, StatusBanner, StatusCard
  tab_config.py  # Form scrollável, Procurar pasta, barra Salvar/Testar/Importar
  tab_exec.py    # Card SCM, Controle vs Administração
  tab_logs.py    # Toolbar, Text monospace, auto-refresh opcional
app.py           # Controller: repo, serviço, callbacks (sem layout pesado)
```

**Regra:** `app.py` (ou `*_app.py`) só orquestra; layout fica em `ui/`.

## Layout obrigatório (shell)

```text
┌─────────────────────────────────────────────────────────┐
│ [ícone]  Título do produto          [chip Dev|Prod]     │
│          Subtítulo (Integração X → Apex)                │
├─────────────────────────────────────────────────────────┤
│  [ Configuração ] [ Execução ] [ Logs ]                 │
│  … conteúdo da aba …                                    │
├─────────────────────────────────────────────────────────┤
│ Config: …\config.db          (clique copia caminho)     │
└─────────────────────────────────────────────────────────┘
```

- **Chip:** `Desenvolvimento` (`%LOCALAPPDATA%\…Dev\`) vs `Produção` (`%PROGRAMDATA%\…`).
- Janela ~920×680, `minsize` ~860×560.

## Abas — comportamento comum

### Configuração

- Scroll vertical para formulários longos.
- Seções com título + separador (não empilhar muitos `LabelFrame` sem hierarquia).
- Pastas: botão **Procurar…** (`filedialog.askdirectory`).
- Secrets: vazio = manter; **Limpar** = apagar no próximo salvar.
- Rodapé da aba: **Salvar** (`Primary.TButton`), **Testar conexão Apex**, **Importar .env**.
- **StatusBanner** para feedback (ok / erro / neutro).

### Execução

- **StatusCard** do serviço Windows (nome SCM fixo por produto).
- Grupo **Controle:** Atualizar, Iniciar, Parar, Reiniciar.
- Grupo **Administração:** Instalar (auto-start), Remover — copy sobre elevação.
- Polling de status ~3 s (`after`) — subprocess com **`CREATE_NO_WINDOW`** (ver abaixo).

### Logs

- Tail de `monitor.log` (últimas ~500 linhas).
- **Recarregar**, **Abrir pasta**, checkbox **Auto-atualizar** (ex.: 5 s, só com aba Logs visível).

## Estilos ttk (`ui/theme.py`)

Registrar após `sv_ttk.set_theme`:

| Style | Uso |
|-------|-----|
| `Primary.TButton` | Salvar, Iniciar |
| `Danger.TButton` | Remover serviço |
| `Muted.TLabel` | Ajuda, footer |
| `Section.TLabel` | Títulos de seção |
| `StatusOk / Warn / Err / Muted.TLabel` | Card de serviço |
| `Accent.TLabel` | Chip Dev/Prod |

## Serviço Windows e subprocess

- **Dois executáveis:** `Monitor*Config.exe` (GUI, `console=False`) + `Monitor*Service.exe` (motor SCM).
- **Não** abrir o exe do serviço com duplo clique — erro 1063; instalar via UI ou `*Service.exe install`.
- Em `service_control._run`: **`subprocess.CREATE_NO_WINDOW`** em todo `sc query/start/…` para evitar flash de CMD com a UI aberta.
- Instalação do SCM deve apontar para **`*Service.exe`** na mesma pasta do Config (não o Config.exe).

## PyInstaller (Config)

No `.spec` do Config:

- `datas`: `collect_data_files('sv_ttk')`, `collect_data_files('tzdata')`, `assets/icon.png` (+ `.ico` se necessário).
- `EXE(..., icon='assets/icon.ico', console=False)`.
- `hiddenimports`: `sv_ttk`, `win32timezone`, `zoneinfo`.

Service spec: entrada `monitor_service.py`, `console=True` (debug/install via pywin32), sem Tk/sv-ttk.

## O que customizar por client

| Item | Exemplo SOCOR | Exemplo HP | Exemplo Tasy (futuro) |
|------|-----------------|------------|------------------------|
| `APP_TITLE` | Monitor SOCOR | Monitor HP | Monitor Tasy |
| `APP_SUBTITLE` | Integração MV → Apex | Integração HP → Apex | Integração Tasy → Apex |
| Ícone / cores accent | Teal MV | (definir) | (definir) |
| Nome SCM / pasta ProgramData | `MonitorSocor` | `MonitorHP` | `MonitorTasy` |
| Campos da aba Config | pastas MV, intervalos | pastas HP, retenção 24h | conforme negócio |
| `monitor/` + `monitor_core.py` | parser `*_alerta.txt` | HTML HP | (a definir) |

**Igual entre clients:** shell, abas, tema, banners, card de serviço, padrão SQLite+DPAPI, teste Apex service client, Ruff, estrutura de testes.

## Checklist — portar UI para outro repo (HP / Tasy)

1. Copiar pacote `ui/` e adaptar `branding.py` (título, subtítulo, opcionalmente accent).
2. Refatorar `app.py` para controller + views (como SOCOR).
3. Adicionar `sv-ttk` e `pyproject.toml` (Ruff).
4. Criar `assets/` + `scripts/generate_icon.py` (ou reutilizar script).
5. Atualizar `*Config.spec` (sv-ttk + ícones).
6. Garantir `service_control._subprocess_kwargs` / `CREATE_NO_WINDOW`.
7. `resolve_service_executable()` + install apontando para `*Service.exe`.
8. Testes: smoke `test_ui_theme.py`, suite existente, `ruff check .`.
9. Atualizar [[Projetos/monitor-hp/Visao Geral|Visão Geral]] / doc do client com link para **esta nota**.

## Adoção atual

| Client | UI padrão |
|--------|-----------|
| [[Projetos/monitor-socor/Visao Geral|monitor-socor]] | Implementado (referência) |
| [[Projetos/monitor-hp/Visao Geral|monitor-hp]] | Tkinter legado — **pendente** alinhamento |
| Tasy | A definir |

## Links

- Implementação: `C:\Estudos\monitor-socor\ui\`
- Operacional SOCOR: [[Projetos/monitor-socor/monitor-socor - Setup e operação|Setup SOCOR]]
- Apex ingest: [[Projetos/apex/API - Guia Novos Clients|Guia Novos Clients]]
