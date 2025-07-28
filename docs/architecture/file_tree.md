---
title: Estrutura de Diretórios – LuxorASAP (produção 2025)
tags: [architecture, file-tree]
---

# Visão geral

Este documento apresenta a **árvore de diretórios completa** do LuxorASAP na versão que está hoje em produção, acompanhada de uma breve explicação de cada item.  
*Legenda*: pastas terminam com `/`; arquivos exibem a **descrição** comentada na mesma linha.

> **Observação** – Esta árvore é a “fonte da verdade” para localizar artefatos.  
> Scripts de ETL e módulos Python têm documentação técnica adicional em `docs/modules/…`.

## Árvore de diretórios detalhada

```text
LuxorASAP
├─ carteira_online/
│  ├─ production/
│  │  ├─ bases_input/
│  │  │  ├─ Operation_Desk_V4.xlsm      – planilha p/ ajuste de posição; CRUD de ativos (1 aba/fundo)
│  │  │  └─ input_data.xlsx             – cópia gerada ao clicar “salvar input”; lida por portfolio_builder.py
│  │  ├─ carteiras_btg/                 – planilhas BTG on-shore; lidas por leitor_carteira_excel.py
│  │  ├─ bases_historicas/
│  │  │  └─ historical_return.xlsx      – cotas gerenciais geradas por return_calculator.py
│  │  ├─ bases_output/
│  │  │  └─ base_portfolios.xlsx        – carteira gerencial; cópia diária vai para carteiras_luxor_historico/
│  │  ├─ carteiras_luxor_historico/
│  │  │  ├─ 2025/                       – relatórios de abertura/fechamento diários
│  │  │  ├─ bases_completas/            – histórico completo das carteiras
│  │  │  └─ metricas_risco/
│  │  │      └─ hist_risk_metrics.xlsx  – métricas de risco históricas
│  │  ├─ portfolio_builder.py           – atualiza base_carteira_online_v12.xlsx + métricas
│  │  ├─ return_calculator.py           – calcula cotas diárias, retornos 1D/MTD/YTD
│  │  ├─ asset_data_extraction.py       – legacy – importado mas não mais usado
│  │  ├─ btg_data_extraction.py         – lê carteiras_btg.xlsx; usado em cash_calculator.py
│  │  ├─ cash_calculator.py             – quebra de caixa & provisão de trades a liquidar
│  │  ├─ historical_data_extractor.py   – legacy – swap NDF (parou em 2023)
│  │  ├─ leitor_carteira_excel.py       – compila carteiras BTG mais recentes
│  │  └─ onedrive_file_redirect.py
│  ├─ base_carteira_online_v12.xlsx     – output final p/ PBI carteira_online.pbix
│  └─ historical_return.xlsx            – output final p/ PBI relatorio_retornos.pbix
├─ source_bases/
│  ├─ scripts/
│  │  ├─ hist_concentration_updater.py  – atualiza hist_portfolios_concentration.parquet (descontinuado)
│  │  ├─ custom_funds_quotas_updater.py – gera cotas p/ portfolios customizados
│  │  ├─ non_bbg_data_updater.py        – cria hist_non_bbg_px_last.parquet
│  │  ├─ daily_pnl_updater.py           – migração p/ luxor-data-pipelines (não rodar com --hist aqui)
│  │  ├─ historical_quotas_updater.py   – atualiza all_funds_quotas.parquet
│  │  ├─ risk_metrics_updater.py        – incremental historico_risk_metrics.xlsx → parquet
│  │  ├─ us_cash_updater.py             – gera hist_us_cash.parquet (caixa offshore)
│  │  └─ swaps_updater.py               – legacy – swaps zerados em 2023
│  ├─ ativos_sem_bbg.xlsx               – séries personalizadas (IPCA+7, Hawker etc.)
│  ├─ historico_caixa.xlsx              – caixa US histórico (possível legado)
│  ├─ historico_risk_metrics.xlsx       – métricas históricas dos portfolios
│  ├─ historico_swap.xlsx               – swap histórico (atualização encerrada em 2023)
│  ├─ historico_trades_ativos.xlsx      – **planilha-mestre** de boletas, ativos, preços ilíquidos, map-fields
│  ├─ manual_prices_inputs.xlsx         – input manual de cotas oficiais (IP Atlas, TCI…)
│  └─ others.xlsx                       – regras de aporte/resgate, taxas, corretagens (PBI operations_report)
├─ run_luxorASAP.py                     – daemon: monitora historico_trades_ativos.xlsx → parquet & posições
│  ├─ fund.py                           – abstração de fundo: ativos, bancos, trades, caixa
│  ├─ asset.py                          – processa trades de um ativo → quantidades por data
│  └─ bank_account.py                   – posições & fluxos por conta bancária
├─ source_bases_updater.py              – orquestra scripts em source_bases/scripts/
├─ market_data_extractor.py             – extrai BDH/BDP do Bloomberg (LuxorDB/raw)
├─ market_data_loader.py                – unifica preços, calcula estimativas e atualiza LuxorDB/tables
├─ luxorDB_datareader.py                – API de consulta ao LuxorDB
├─ luxorDB_dataloader.py                – camada de persistência padronizada p/ LuxorDB
└─ LuxorDB/
   ├─ raw/                              – dumps brutos (Bloomberg, etc.)
   └─ tables/                           – tabelas processadas (níveis variados)
```