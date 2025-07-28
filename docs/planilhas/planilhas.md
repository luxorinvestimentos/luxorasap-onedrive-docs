---
title: Planilhas Detalhadas
tags: [spreadsheets]
---

# Planilhas Detalhadas

Esta página descreve em detalhes as principais planilhas utilizadas pelo **LuxorASAP**, indicando sua localização, função e scripts que interagem com cada uma delas.

---

## Operation_Desk_V4.xlsm

- **Localização**  
  `carteira_online/production/bases_input/Operation_Desk_V4.xlsm`

- **Descrição**  
  Planilha de cálculo de ajustes de posição e CRUD de ativos da carteira online. Possui uma aba dedicada para cada fundo, listando seus ativos e quantidades.

- **Interações Principais**  
  - `portfolio_builder.py`: lê as abas para montar o portfólio inicial.  
  - Serve de base para geração de `input_data.xlsx`.  
  - Influencia o `cash_calculator.py` através das posições calculadas.

---

## input_data.xlsx

- **Localização**  
  `carteira_online/production/bases_input/input_data.xlsx`

- **Descrição**  
  Cópia da `Operation_Desk_V4.xlsm` gerada ao clicar no botão "salvar input". Contém abas para cada fundo com as quantidades e uma aba `ativos` com o cadastro de ativos.

- **Interações Principais**  
  - `portfolio_builder.py`: usa para gerar `base_portfolios.xlsx` e atualizar `base_carteira_online_v12.xlsx`.  
  - Alimenta o relatório Power BI `carteira_online.pbix`.

---


## base_portfolios.xlsx

- **Localização**  
  `carteira_online/production/bases_output/base_portfolios.xlsx`

- **Descrição**  
  Base gerencial sem agregações, com posições e quantidades por fundo, gerada ao final do pregão pelo `portfolio_builder.py`.

- **Interações Principais**  
  - `portfolio_builder.py`: cria e salva nesta pasta.  
  - Copiada para `carteiras_luxor_historico/bases_completas/` diariamente.  
  - `risk_metrics_updater.py` e `hist_concentration_updater.py`: consomem para cálculo de métricas.

---

## base_carteira_online_v12.xlsx

- **Localização**  
  `carteira_online/base_carteira_online_v12.xlsx`

- **Descrição**  
  Output final do `portfolio_builder.py`, com tabelas prontas para o relatório Power BI `carteira_online.pbix`.

- **Interações Principais**  
  - `portfolio_builder.py`: gera esta planilha.  
  - Power BI (`carteira_online.pbix`): consome diretamente.

---

## historical_return.xlsx (raw)

- **Localização**  
  `carteira_online/production/bases_historicas/historical_return.xlsx`

- **Descrição**  
  Base de cotas gerenciais calculadas pelo `return_calculator.py`. Precisa de ajustes manuais nas quantidades para refletir aplicações e resgates dos fundos.

- **Interações Principais**  
  - `return_calculator.py`: gera esta planilha.  
  - `historical_quotas_updater.py`: consome para atualizar séries de cotas em `source_bases`.  
  - Relatório Power BI `relatorio_retornos.pbix`.

---

## historical_return.xlsx (output final)

- **Localização**  
  `carteira_online/historical_return.xlsx`

- **Descrição**  
  Versão final das tabelas de retornos gerenciais (1D, MTD, YTD) calculadas pelo `return_calculator.py` e formatadas para o relatório Power BI `relatorio_retornos.pbix`.

- **Interações Principais**  
  - `return_calculator.py`: gera esta cópia final.  
  - Power BI (`relatorio_retornos.pbix`): consome diretamente.

---

