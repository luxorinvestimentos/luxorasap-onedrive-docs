# 🧠 LuxorASAP

**Luxor Automatic System for Assets and Portfolios** é o toolbox oficial da Luxor para manipulação de dados de investimento, integração com provedores externos (BTG Pactual), e gerenciamento de pipelines de dados para análise, marcação e reporting.

Projetado para automatizar tarefas recorrentes do time de gestão e analytics, o LuxorASAP provê uma interface limpa, extensível e segura para acessar:

- ✅ Tabelas financeiras em Parquet
- ✅ API de relatórios do BTG Pactual
- ✅ Registro e leitura de boletas offshore
- ✅ Gravação incremental no ADLS (Azure Blob Storage)
- ✅ Utilitários para DataFrames e schemas

---

## 📦 Estrutura de Módulos

- [`btgapi`](luxorasap/btgapi.md): Integração autenticada com a API do BTG
- [`datareader`](luxorasap/datareader.md): Consulta a dados de mercado e fundos
- [`ingest`](luxorasap/ingest.md): Salvamento e atualização de tabelas no ADLS
- [`utils`](luxorasap/utils.md): Leitura de arquivos binários e acesso ao blob

---

## 🔑 Autenticação e Ambiente

Algumas funcionalidades exigem credenciais armazenadas de forma segura. O acesso ao Blob Storage e à API do BTG requer:

### ✅ Variáveis de ambiente (recomendado)

Crie um arquivo `.env` na raiz do seu projeto com o seguinte conteúdo:

```bash
# Azure Blob Storage
AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=...;AccountName=..."

# BTG API
BTG_CLIENT_ID="seu_client_id"
BTG_CLIENT_SECRET="seu_client_secret"
```

### 🧪 Alternativa

Você também pode passar os parâmetros diretamente nas funções:

```python
get_access_token(client_id="...", client_secret="...", test_env=False)
```

---

## 🛠️ Exemplo de uso

```python
from luxorasap.datareader import LuxorQuery
from luxorasap.ingest import save_table
from luxorasap.btgapi import get_access_token, request_portfolio

lq = LuxorQuery()
df = lq.get_table("trades")
save_table("trades_backup", df)

token = get_access_token()
ticket = request_portfolio(token, "FUNDO XPTO", start_date, end_date)
```

---

## 📄 Licença

Este projeto é de uso interno da Luxor Investimentos.