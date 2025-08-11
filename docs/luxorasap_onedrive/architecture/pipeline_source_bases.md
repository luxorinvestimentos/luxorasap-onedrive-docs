# Arquitetura: Pipeline `source_bases`

Este documento detalha o pipeline executado pelo script `source_bases_updater.py`, que atua como orquestrador para a atualização de diversas bases de dados auxiliares utilizadas pelo LuxorASAP.

---

## 1. Visão Geral do Pipeline

O `source_bases_updater.py` executa uma série de scripts especializados, cada um responsável por gerar ou atualizar um conjunto específico de dados em formato `.parquet` no ADLS, no diretório `luxorasap/enriched/parquet`. Embora sejam independentes, os scripts são executados sequencialmente.

<div align="center">
```mermaid
flowchart LR
    subgraph "**Execução em cascata**"
        direction LR
        A[non_bbg_data_updater.py]
        B[historical_quotas_updater.py]
        C[risk_metrics_updater.py]
        D[hist_concentration_updater.py]
        E[us_cash_updater.py]
    end

    subgraph "**luxorasap/enriched/parquet**"
        A -->|dataloader| A1[(hist_non_bbg_px_last.parquet)]
        B -->|dataloader| B1[(all_funds_quotas.parquet)]
        B -->|dataloader| B2[("[NOME_FUNDO]_quotas.parquet")]
        C -->|dataloader| C1[(hist_risk_metrics.parquet)]
        D -->|dataloader| D1[(hist_concentration.parquet)]
        E -->|dataloader| E1[(hist_us_cash.parquet)]
    end
```
</div>

---

## 2. non_bbg_data_updater.py

Fluxo de execução:

<div align="center">
```mermaid
flowchart TB
    A0["non_bbg_data_updater.py"]--> B["run()"]
    A[ativos_sem_bbg.xlsx] -->|input| B["run()"]
    B --> C[update_hawker]
    B --> D[get_fidc_trybeI/II_quote]
    B --> E[Calcular séries adicionais]
    E --> F[(hist_non_bbg_px_last.parquet)]
    E -->|sobrescreve| A
```
</div>
---


O script `non_bbg_data_updater.py` concentra diversos métodos para criação de séries históricas de dados não-Bloomberg. Entre esses:

- Cálculo de séries IPCA+, CPI+, loans (fixos e variáveis).
- Método `update_hawker`: estimativa diária de cotas das séries SPX Hawker durante o mês corrente.
- Métodos `get_fidc_trybeI_quote` e `get_fidc_trybeII_quote`: atualização de cotas dos FIDCs a partir do arquivo `carteiras_btg.xlsx` mais recente.
- Chamada `run()` central coordena tudo.


## 3. historical_quotas_updater.py

Fluxo de execução:

<div align="center">
```mermaid
flowchart TB
    H1["historico_cotas.xlsx"] --> H2["Remove dados D0"]
    R1["historical_return.xlsx"] --> R2["Extrair dados atuais"]
    H2 & R2 --> M["Atualização de cotas e quantidades"]
    M -->|sobrescreve| H1
    M --> S2[("all_funds_quotas.parquet")]
    M --> S3[("[NOME_FUNDO]_quotas.parquet")]
```
</div>


O script `historical_quotas_updater.py` é responsável por atualizar as cotas históricas de todos os fundos Luxor. Ele realiza os seguintes passos:

1. Carrega a planilha `historico_cotas.xlsx` até o dia anterior.
2. Carrega a planilha `carteira_online/production/bases_historicas/historical_return.xlsx` gerada pelo `return_calculator.py`.
3. Remove a última data existente em `historico_cotas.xlsx`.
4. Atualiza `historico_cotas.xlsx` com as cotas e quantidades mais recentes provenientes dos `historical_returns`.
5. Para fundos não presentes em `historical_returns`, propaga cota e quantidade do dia anterior.
6. Salva (sobrescreve) `historico_cotas.xlsx`.
7. Gera e salva no ADLS:
    - `all_funds_quotas.parquet`
    - `[NOME_FUNDO]_quotas.parquet` para cada fundo.



---

## 4. risk\_metrics\_updater.py

Fluxo de execução:
<div align="center">

```mermaid
flowchart TD
    ExcelIn["hist_risk_metrics.xlsx<br>(veio do portfolio_builder)"] -->|input| UpdateRisk["Atualização incremental"]
    ExcelIn2["historico_risk_metrics.xlsx"] -->|input| UpdateRisk
    UpdateRisk -->|sobrescreve| ExcelIn2
    UpdateRisk --> SaveParquet[("hist_risk_metrics.parquet<br>(luxorasap/enriched/)")]
```
</div>
---

O script `risk_metrics_updater.py` simplesmente recebe como input esta planilha:

```
carteira_online/production/carteiras_luxor_historico/metricas_risco/hist_risk_metrics.xlsx
```

Essa planilha possui as métricas mais atualizadas vindas da última execução do `portfolio_builder.py`. Em seguida, realiza:

- Atualização incremental da base `source_bases/historico_risk_metrics.xlsx`.
- Sobrescreve a base `source_bases/historico_risk_metrics.xlsx`.
- Salva no ADLS `hist_risk_metrics.parquet`.



## 5. hist\_concentration\_updater.py

Fluxo de execução:
<div align="center">
```mermaid
flowchart TB
    C1[base_portfolios.xlsx] --> D[Extrair]
    C2[10 últimas bases_completas] --> D
    D --> E[Agregação]
    E --> F[(hist_concentration.parquet)]
```
</div>

Este script atualiza a base `hist_concentration.parquet` a partir da **`base_portfolios.xlsx`** e dos **10** arquivos mais recentes em `bases_completas/`.
> 🛈 **Uso atual baixo** – Este fluxo está sendo pouco usado, vale avaliar se pode ser descontinuado.
---


## 6. us_cash_updater.py

Fluxo de execução:

<div align="center">
```mermaid
flowchart TB
    U1[base_portfolios.xlsx] --> V[Extrair caixa]
    U2[bases_completas] --> V
    V --> W[("hist_us_cash.parquet<br>(luxorasap/enriched/)")]
```
</div>

O script `us_cash_updater.py` atualiza o histórico de caixa dos fundos A, B e HMX em:

```
luxorasap/enriched/parquet/hist_us_cash.parquet
```

Fontes:

- Planilha `base_portfolios.xlsx` mais recente (gerada pelo `portfolio_builder.py`).
- Arquivos históricos em `carteiras_luxor_historico/bases_completas/`.

