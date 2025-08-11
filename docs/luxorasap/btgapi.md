# Módulo `btgapi`

Este módulo oferece um wrapper autenticado para integração com as APIs do BTG Pactual, incluindo relatórios, boletas offshore e autenticação JWT.

---

## 📌 Autenticação

### `get_access_token`

Obtém um token JWT válido (~1h) para autenticação nas APIs do BTG.

```python
from luxorasap.btgapi import get_access_token

token = get_access_token(test_env=True)
```

**Parâmetros:**

- `client_id` (`str`, opcional): ID do cliente. Se `None`, será lido de variável de ambiente.
- `client_secret` (`str`, opcional): Segredo do cliente. Se `None`, será lido de variável de ambiente.
- `test_env` (`bool`): Define se usa ambiente de testes. Default: `True`.
- `timeout` (`int`): Timeout em segundos. Default: `20`.

**Retorna:** `str` — Token de autenticação.  
**Erros:** `BTGApiError`

---

## 📑 Relatórios

### `request_portfolio`

Solicita relatório de carteira para um fundo.

```python
ticket = request_portfolio(token, "FUND NAME", start_date, end_date)
```

**Parâmetros:**
- `token` (`str`): Token JWT.
- `fund_name` (`str`)
- `start_date`, `end_date` (`datetime.date`)
- `format` (`str`): `"excel"`, `"xml5"` ou `"pdf"`

**Retorna:** `str` — Ticket da requisição.

---

### `check_report_ticket`

Verifica se um ticket foi processado.

**Retorna:** `bytes` se o conteúdo estiver pronto.  
**Erros:** `BTGApiError` em caso de pendência ou erro.

---

### `await_report_ticket_result`

Aguarda até que o ticket esteja pronto e retorna conteúdo binário (`bytes`).

```python
zip_bytes = await_report_ticket_result(token, ticket)
```

---

### `process_zip_to_dfs`

Extrai todos os arquivos `.zip` e retorna `dict[str, pd.DataFrame]`.

```python
dfs = process_zip_to_dfs(zip_bytes)
```

---

### `request_investors_transactions_report`

Solicita relatório de movimentações por cotistas (RTA).

---

### `request_fundflow_report`

Gera relatório FundFlow (RTA) com base em datas e fundo.

```python
ticket = request_fundflow_report(token, start_date, end_date, fund_name="FUND")
```

---

## 📥 Trades Offshore

### `submit_offshore_equity_trades`

Submete lista de trades offshore (equities).

```python
ticket = submit_offshore_equity_trades(token, trades=[{...}], test_env=True)
```

---

### `await_transaction_ticket_result`

Consulta e espera o resultado de uma boleta enviada.

```python
df = await_transaction_ticket_result(token, ticket)
```

**Retorna:** `pd.DataFrame` com status, ticket, referência, mensagens etc.

---

## ⚠️ Exceções

### `BTGApiError`

Exceção customizada lançada em qualquer erro de autenticação, requisição ou processamento.