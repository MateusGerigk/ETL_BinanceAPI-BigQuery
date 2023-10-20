
# ETL - BINANCE API E BIGQUERY

Script ETL para extração de registro de valores de uma das maiores corretoras de criptomoedas mundial, baseado em índice da moeda e intervalo de tempo, onde foram realizados alguns tratamentos de dados em python e posteriormente enviado a Google Cloud Plataform para manipulação através da ferramenta BIGQUERY e visualização em gráficos através da ferramenta LOOKER STUDIO.


## Ferramentas e Bibliotecas

[Binance API](https://www.binance.com/en-AU/binance-api) - API utilizada para extração de dados da Corretora;

[Google BigQuery](https://console.cloud.google.com/projectselector2/bigquery) - Armazenamento em nuvem de dataframe e manipulação SQL;

[Google Looker Studio](https://lookerstudio.google.com/u/0/) - Exibição dos dados em gráficos;

[Python-Binance](https://python-binance.readthedocs.io/en/latest/) - Biblioteca para comunicação com API Binance;

[Google.Cloud](https://cloud.google.com/python/docs/reference?hl=pt-br) - Biblioteca cliente para APIs do Cloud (Utilizada para comunicação com BigQuery); 

[Google.Oauth2](https://developers.google.com/identity/protocols/oauth2/scopes?hl=pt-br) - Biblioteca de autenticação para comunicação com APIs Google.

[OS](https://docs.python.org/pt-br/3/library/os.html) - Pacote para utilização de variáveis de ambiente para ocultação de KEY e SECRET KEY da API Binance, geradas através da conexão com a conta da corretora.

[Pandas](https://pandas.pydata.org) - Biblioteca para manipulação e tratamento de dados através de dataframes(CSV).

[NumPy](https://numpy.org) - Biblioteca para manipulação e transformação de dados recebidos através de chamado para API Binance.


## Variáveis de Ambiente

Para rodar esse projeto, você vai precisar adicionar as seguintes variáveis de ambiente no seu .env

`BINANCE_API_KEY`

`BINANCE_SECRET_KEY`




##  Utilização do Script
### Importação de bibliotecas
```bash
import pandas as pd
import numpy as np
from binance.client import Client
import os
from google.cloud import bigquery
from google.oauth2 import service_account
```

### Conexão Big Query

Para que a conexão com a GCP ocorra normalmente e possa ser utilizada as plataformas BigQuery e Looker Studio, é necessário que o usuário crie um projeto no painel [IAM ADMIN](https://console.cloud.google.com/iam-admin/iam) para concessão de acessos para usuários em bases/arquivos recebidos de forma externa.

Apóx isso é necessária acessar o menu *Contas de Serviço* e criar uma nova conta vinculada ao projeto, gerenciar as chaves de acesso da conta e criar uma nova chave JSON, onde o arquivo deverá ser anexado ao diretório do SCRIPT ETL.

Com o processo de emissão da chave JSON de conexão com o GCP finalizada, basta criar a variável *credentials* de conexão. 


```python
credentials = service_account.Credentials.from_service_account_file("***DIRETÓRIO ARQUIVO CHAVE JSON****",
                                                                    scopes=["https://www.googleapis.com/auth/cloud-platform",
                                                                            "https://www.googleapis.com/auth/drive"])
```

### Conexão API Binance

```python
api_key = os.getenv(key = 'BINANCE_API_KEY')
api_secret = os.getenv(key = 'BINANCE_SECRET_KEY')
client = Client(api_key, api_secret)
```
## Extração dos dados

 - O arquivo proveniente da extração é recebido como array, através da chamada da função passando apenas o SIMBOLO do par de criptomoedas que são negociadas na corretora. 
- O intervalo por padrão e a data de início da disponibilização dos dados são definidos dentro da função.
 - A função realiza a extração dos arquivos em array, forçando a transformação do mesmo.

 ## Transformação dos dados
 - Os dados são transformados em dataframe através da biblioteca Pandas, onde é atribuida os índices das colunas e conversão da coluna do tipo data. 
 - Criação das colunas *Media Movel, Desvio Padrão, Banda Superior e Banda Inferior* para criação do índice BANDAS DE BOLLINGER, utilizado para tendência de alta ou baixa de preços no mercado financeiro.

 ## Carregamento dos dados

 - Carregamento do arquivo CSV para o GCP BigQuery através da API Google, utilizando o comando: 
    ```SIMBOLO.to_gbq(credentials=credentials, destination_table='PROJETOGCP.SIMBOLO', if_exists='replace')```

 ## Visualização dos dados

 - Visualização dos gráficos CANDLESTICKS e BANDAS DE BOLLINGER através da ferramenta Google Looker Studio:
![Logo](https://imgur.com/r2EaOUB.png)

