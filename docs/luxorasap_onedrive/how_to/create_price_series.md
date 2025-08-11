# Adicionar Série de Preços

Se a série de preço estiver disponível no Bloomberg, adicione um novo ativo colocando na coluna `Ticker_BBG` a chave referente à série.
Nesse caso, a série passará a integrar o fluxo de atualização explicado no [Bloomberg Data Pipeline](../architecture/bloomberg_data_pipeline.md).

Quando não possuir, será necessário criar uma série de preços para ser consultada com o `Ticker` cadastrado para o ativo.


-   **Ativos Ilíquidos**: preencha o preço na aba `VCs_Prices` da planilha `historico_trades_ativos.xlsx`.
    ```mermaid
    flowchart
      subgraph **Iliquids Prices**
        direction LR
        A@{ shape: lean-r, label: "historico_trades_ativos.xlsx<br>Aba VCs_Prices"} --> B[run_LuxorASAP.py]
        B --> C[("hist_vc_px_last<br>(enriched/parquet)")]
        C -->|"**market_data_loader.py**"| E[("px_last<br>(enriched/parquet)")]
      end
    ```
    *   O script `run_LuxorASAP.py` irá atualizar a tabela `hist_vc_px_last`, o que irá triggar o `market_data_loader.py` para atualizar a tabela `px_last`.



-   **Benchmarks ou séries customizadas** (ex.: IPCA + 7%, CDI + 1%):
    ```mermaid
    flowchart
      subgraph **Non BBG Prices**
        direction LR
        F@{ shape: lean-r, label: "ativos_sem_bbg.xlsx<br>(nova coluna)"} --> G["non_bbg_data_updater.py<br>(lógica de criação)"]
        G --> H[("hist_non_bbg_px_last<br>(enriched/parquet)")]
        H -->|"**market_data_loadey.py**"| I[("px_last<br>(enriched/parquet)")]
      end
    ```
    *   Adicione uma coluna na planilha `source_bases/ativos_sem_bbg.xlsx`.
    *   Implemente a lógica de criação da série em `source_bases/scripts/non_bbg_data_updater.py`.
    *   *Observação para séries do Hawker (até 31/07/2025)*: O preço também está sendo definido aqui. Não precisa inserir a coluna manualmente na planilha nesse caso, mas configure o novo ticker dentro do script `non_bbg_data_updater.py`, seguindo o padrão das anteriores.
    *   A execução do `non_bbg_data_updater.py` irá atualizar a tabela `hist_non_bbg_px_last`, o que irá triggar o `market_data_loader.py` para atualizar a tabela `px_last`.
    > **Sugestão de Melhoria**: Migrar o fluxo de atualização das cotas do Hawker e FIDCs para o input manual abaixo.

    
-   **Input manual**: adicione o preço na planilha `source_bases/manual_prices_input.xlsx`.
    ```mermaid
    flowchart TD
      subgraph **Manual Prices**
        direction LR
        K@{shape: lean-r, label: "manual_prices_input.xlsx"} -->|"market_data_loadey.py"| a[("manual_prices_input (raw/parquet)")]
        a -->|"**market_data_loader.py**"| b[("custom_prices<br>(enriched/parquet)")]
        b -->|"**market_data_loader.py**"| c["px_last<br>(enriched/parquet)"]
      end
    ```
    *   A atualização desse Excel irá triggar o `market_data_loader.py` a salvar a tabela `manual_prices_input` dentro de `luxorasap/raw/parquet/`.
    *   Isso, por sua vez, irá triggar o próprio `market_data_loader.py` a atualizar a base `custom_prices` (nessa etapa, o portfolio do TCI será rodado, estimando as cotas até a data de hoje, e as cotas do IP Atlas também serão atualizadas até hoje usando o SP500).
    *   Finalmente, isso irá triggar uma atualização da base `px_last`.
    > **Sugestão de Melhoria**: tirar do `market_data_loader.py` a parte do código que faz o cálculo de preços do TCI e IP Atlas, deixando um script separado para centralizar
    > esses cálculos. Para esse script, migrar também a lógica para estimar as cotas do Hawker que está implementada no `non_bbg_data_updater.py`.
    
    