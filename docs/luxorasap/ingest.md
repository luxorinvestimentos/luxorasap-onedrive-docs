# Módulo `ingest`

O módulo `ingest` é responsável por gravar e atualizar tabelas no ADLS. Ele oferece uma API moderna (`cloud`) e uma interface legada (`legacy_local.DataLoader`), mantida apenas para compatibilidade.

---

## ☁️ Interface Moderna: `cloud`

### `save_table`

Salva um `DataFrame` como Parquet no Azure Blob Storage.

```python
from luxorasap.ingest import save_table

save_table("trades", df)
```

**Parâmetros:**

- `table_name` (`str`): Nome da tabela (sem extensão).
- `df` (`pd.DataFrame`): Dados a serem salvos.
- `index` (`bool`): Se `True`, inclui o índice. Default: `False`.
- `index_name` (`str`): Nome da coluna de índice (se aplicável). Default: `"index"`.
- `normalize_columns` (`bool`): Se `True`, normaliza o nome das colunas.
- `directory` (`str`): Caminho no ADLS. Default: `"enriched/parquet"`.

---

### `incremental_load`

Atualiza a tabela existente no ADLS com novos dados, preservando históricos e removendo duplicações com base em uma coluna incremental.

```python
from luxorasap.datareader import LuxorQuery
from luxorasap.ingest import incremental_load

incremental_load(lq, "prices_daily", df_new, increment_column="Date")
```

**Parâmetros adicionais:**

- `lq` (`LuxorQuery`): Para acessar a versão atual da tabela.
- `increment_column` (`str`): Nome da coluna que define o corte temporal.

---

## 🗂️ Interface Legada: `legacy_local.DataLoader`

> ⚠️ **Deprecação em andamento:** prefira a API `cloud`.

```python
from luxorasap.ingest import DataLoader

dl = DataLoader(luxorDB_directory=Path("/tmp/tabelas"))
```

### Recursos principais:

- `add_file_tracker(...)`: Adiciona arquivo Excel para monitoramento e carga.
- `load_file_if_modified(...)`: Recarrega arquivo se houver nova versão.
- `load_table_if_modified(...)`: Substitui tabela se o timestamp for mais recente.
- `scan_files()`: Varre e processa todos os arquivos monitorados.
- Suporte a exportação para Parquet + ADLS

---

## 🧠 Observações Técnicas

- O cliente padrão de escrita é `BlobParquetClient` via `utils.storage`
- Os dados são convertidos para `str` antes do salvamento
- Suporte parcial a leitura via `bytes` (para APIs ou uploads)