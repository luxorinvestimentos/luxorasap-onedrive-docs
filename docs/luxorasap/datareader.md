# Módulo `datareader`

Este módulo provê a interface `LuxorQuery` para leitura de tabelas, séries temporais e análises financeiras dentro do data lake da Luxor (ADLS).

---

## 🔍 Classe `LuxorQuery`

### Instanciação

```python
from luxorasap.datareader import LuxorQuery

lq = LuxorQuery()
```

---

## 📄 Tabelas

### `get_table(name: str) -> pd.DataFrame`

Lê uma tabela Parquet da camada enriquecida do ADLS.

```python
df = lq.get_table("trades")
```

### `table_exists(name: str) -> bool`

Verifica se a tabela está registrada no catálogo.

---

## 📈 Preços

### `get_price(asset: str, px_date: str | datetime) -> float | None`

Obtém o preço de fechamento pontual de um ativo em uma data.

```python
price = lq.get_price("msft us equity", px_date="2024-12-31")
```

### `get_prices(asset: str, previous_date, recent_date, period: str=None) -> pd.Series`

Retorna série de preços para o ativo. Period pode ser mtd, ytd, qtr, 1m, 2m, 3m, etc.

```python
series = lq.get_prices("msft us equity", recent_date=datetime.date(2024,12,31), period='ytd')
```

---

## 📊 Análises

### `get_pct_change(asset: str, previous_date, recent_date) -> float`

Retorna a variação percentual no período.

### `get_pct_changes(asset: str, previous_date, recent_date, period: str) -> pd.Series`

Retorna a série de retornos diários. Period pode ser mtd, ytd, qtr, 1m, 2m, 3m, etc.

---

### `calculate_volatility(asset: str, previous_date, recent_date) -> float`

Volatilidade anualizada de retornos diários (std * sqrt(252)).

---

### `calculate_price_correlation(asset1, asset2, previous_date, recent_date) -> float`

Correlação entre os retornos de dois ativos no período.

---

## 💼 Fundos e Carteiras

### `get_positions_and_movements(fund: str, date) -> pd.DataFrame`

Devolve posição do fundo com compras/vendas no dia.

---

### `get_fund_aum(fund: str, date:  datetime.date) -> float`

Retorna o AUM (Assets Under Management) de um fundo numa data.

---

### `get_fund_pnl(fund: str, previous_date: datetime.date, recent_date: datetime.date) -> pd.Series`

Série com o lucro/prejuízo do fundo no período.

---

### `run_return_analysis(asset_or_df, ...) -> dict`

Análise de performance: CAGR, drawdown, sharpe, etc.

---

## 🧠 Outros

- Datas devem ser `datetime.date`
- Preços e retornos sempre assumem coluna `"Price"` por padrão