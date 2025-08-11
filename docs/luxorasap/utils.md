# Módulo `utils`

O módulo `utils` reúne funções auxiliares reutilizáveis, organizadas em dois submódulos principais: `dataframe` e `storage`.

---

## 📦 `utils.dataframe`

Utilitários para padronização e leitura de dados.

---

### `prep_for_save`

Prepara um DataFrame para salvamento consistente.

```python
from luxorasap.utils.dataframe import prep_for_save

df = prep_for_save(df, index=False, normalize=True)
```

**Parâmetros:**
- `df` (`pd.DataFrame`)
- `index` (`bool`): Se `True`, inclui o índice como coluna.
- `index_name` (`str`)
- `normalize` (`bool`): Se `True`, normaliza tudo que for texto para lowercase.

---

### `persist_column_formatting`

Preserva formatações e tipos de colunas de DataFrames Excel.

---

### `read_bytes`

Detecta e lê conteúdo binário como Excel, CSV ou Parquet.

```python
from luxorasap.utils.dataframe import read_bytes

df = read_bytes(content_bytes, filename="arquivo.xlsx")
```

---

## ☁️ `utils.storage`

Interface para leitura/escrita no Azure Blob Storage com Parquet.

---

### Classe `BlobParquetClient`

Cliente que centraliza operações com dados no formato Parquet no ADLS.

#### Métodos principais:

- `write_df(df, blob_path)`
  - Grava o DataFrame no caminho do blob especificado.
- `read_df(blob_path) -> pd.DataFrame`
  - Lê o conteúdo Parquet do blob e retorna como DataFrame.
- `exists(blob_path) -> bool`
  - Verifica se o arquivo existe no blob.
- `list_paths(prefix=None) -> list[str]`
  - Lista todos os arquivos em um diretório lógico no ADLS.

---

## 🧠 Notas

- O `read_df` internamente usa `read_bytes` para detectar o formato.
- Toda escrita em Parquet assume `df.astype(str)` como padrão.