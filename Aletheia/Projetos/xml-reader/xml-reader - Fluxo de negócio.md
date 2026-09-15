---
tags:
  - xml-reader
  - negocio
  - integracao
projeto: xml-reader
ultima-revisao: 2026-09-15
---

# xml-reader — Fluxo de negócio

Relacionado: [[xml-reader]] · [[xml-reader - Documentação técnica]]

## Objetivo

Carregar exames do **Smart** e do **XML HP**, correlacionar pelo código do exame Pardini e **exibir apenas divergências de formato**.

## Pipeline (`lib/exams.ts` → `getComparativoExams`)

```
Promise.all([ fetchHpExams(), fetchSmartExams() ])
  → filtrarExamesMaisRecentes(HP)
  → agruparECompararExames(Smart, HP)
  → filtrarExamesComFormatoDivergente(...)
  → Agrupado[]
```

Implementação das regras puras: `app/_function/functions.ts` (cobertas por `tests/functions.test.ts`).

## Cruzamento Smart × HP

1. Cada linha Smart (`ExamRP`) tem `Cod_exame_Pardini`.
2. No HP, `CodExmApoio` vem como `prefixo|codigo`; compara-se **`split("|")[1]`** com `Cod_exame_Pardini`.
3. Um exame Smart (`smk_cod`) pode ter **vários materiais** (`amo_qlf_rot`); cada material vira entrada em `Agrupado.materiais`.
4. Se existirem várias liberações HP para o mesmo `CodExmApoio`, permanece a de **maior** `UltimaLiberacao` (`filtrarExamesMaisRecentes`).

## Regra de divergência (`app/utils/types.ts`)

```ts
isFormatoDivergente(smartFormato, hpFormato):
  - smartFormato === null  → divergente
  - Number(smartFormato) !== hpFormato → divergente
```

Um exame entra na lista se **qualquer material** tiver **qualquer** `hp_info` que satisfaça a regra acima.

## Filtros SQL (Smart)

Exames ativos de apoio HP, tipo serviço:

- `smk_str = 'HP'`
- `smk_tipo = 'S'`
- `smk_status = 'A'` (e também `<> 'I'` na query)

Tabelas: `smk` → `ams` → `exm_hpardini` → `amo` (detalhes em [[xml-reader - Documentação técnica#Modelo de dados]]).

## UI (`/consultaExame`)

1. `fetch("/api/getComparativo?t=" + Date.now(), { cache: "no-store" })`
2. Busca client-side por `smk_cod` ou `smk_nome`
3. Contador: “N exames com formato divergente”
4. `ExamsComparison` exibe cards Smart × HP (datas com `moment`, formato `DD/MM/YYYY`)
