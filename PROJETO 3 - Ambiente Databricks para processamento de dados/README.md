# 🚀 ***Configuração de Databricks para processamento de dados com PySpark e análise de dados utilizando SQL***

## 📖 **Descrição do Projeto:**
Este projeto configura um ambiente de processamento de dados utilizando Databricks para executar transformações e análises em dados de música. O processamento é feito com PySpark, enquanto a análise é conduzida com SQL para identificar padrões de artistas e músicas.


## 🛠️ Ferramentas Utilizadas:
- **Databricks**: Plataforma de processamento de dados na nuvem.
- **CloudFormation**: Ferramenta de infraestrutura como código para criar recursos de nuvem.
- **PySpark**: Framework de processamento distribuído para trabalhar com dados de grandes volumes.


## 📋 **Descrição do Processo**
1. Criação da infraestrutura no Databricks e AWS CloudFormation.
2. Execução de scripts PySpark para ingestão de dados brutos e estruturação dos mesmos.
3. Aplicação de transformações SQL em dados de música para análise.
4. Execução de análises com consultas SQL para identificar padrões nos dados.
5. Automação do pipeline com workflows no Databricks.


## 💻 **Comandos:** 

### 1. **Criação da Infraestrutura:**
   - Criar a Workspace no Databricks.
   - Criar a stack no CloudFormation.
   - Criar um cluster no Databricks com recursos adequados de EC2 (Abrir a Workspace do Databricks, acessar o menu "Compute" e criar um cluster de máquinas EC2 de acordo com o Hardware necessário).

Obs: Será utilizado dados de amostra da própria Databricks

### Arquivo PySpark para criar a estrutura de dados da fonte (pipe1.py)
```python
from pyspark.sql.types import DoubleType, IntegerType, StringType, StructType, StructField

# Caminho para a fonte de dados
file_path = "/databricks-datasets/songs/data-001/"

# Nome da tabela (você escolhe)
table_name = "raw_song_data"

# Caminho do checkpoint para operações intermediárias
checkpoint_path = "/tmp/pipeline_get_started/_checkpoint/song_data"

# Schema (Organização dos dados)
schema = StructType(
  [
    StructField("artist_id", StringType(), True),
    StructField("artist_lat", DoubleType(), True),
    StructField("artist_long", DoubleType(), True),
    StructField("artist_location", StringType(), True),
    StructField("artist_name", StringType(), True),
    StructField("duration", DoubleType(), True),
    StructField("end_of_fade_in", DoubleType(), True),
    StructField("key", IntegerType(), True),
    StructField("key_confidence", DoubleType(), True),
    StructField("loudness", DoubleType(), True),
    StructField("release", StringType(), True),
    StructField("song_hotnes", DoubleType(), True),
    StructField("song_id", StringType(), True),
    StructField("start_of_fade_out", DoubleType(), True),
    StructField("tempo", DoubleType(), True),
    StructField("time_signature", DoubleType(), True),
    StructField("time_signature_confidence", DoubleType(), True),
    StructField("title", StringType(), True),
    StructField("year", IntegerType(), True),
    StructField("partial_sequence", IntegerType(), True)
  ]
)

# Leitura do stream de dados
(spark.readStream
  .format("cloudFiles")
  .schema(schema)
  .option("cloudFiles.format", "csv")
  .option("sep","\t")
  .load(file_path)
  .writeStream
  .option("checkpointLocation", checkpoint_path)
  .trigger(availableNow=True)
  .toTable(table_name)
)

```
### Rodando o script

#Acessar o menu New > Notebook >"Escolher o tipo de linguagem e o cluster > Clicar em Run

Obs: Para inferência do schema automatica podemos utilizar o Auto Loader (Semelhante ao Aws Glue)



### Aplicando transformação aos dados com SQL (pipe2)

Basta criar um novo notebook, alterar a linguagem para SQL e executar o script abaixo:

```sql
CREATE OR REPLACE TABLE
  tb_song_data (
    artist_id STRING,
    artist_name STRING,
    duration DOUBLE,
    release STRING,
    tempo DOUBLE,
    time_signature DOUBLE,
    title STRING,
    year DOUBLE,
    processed_time TIMESTAMP
  );

INSERT INTO
  tb_song_data
SELECT
  artist_id,
  artist_name,
  duration,
  release,
  tempo,
  time_signature,
  title,
  year,
  current_timestamp()
FROM
  raw_song_data

```


### Analytics com Databricks SQL
-- Qual artista publica mais músicas em cada ano?

```sql
SELECT
  artist_name,
  count(artist_name)
AS
  num_songs,
  year
FROM
  tb_song_data
WHERE
  year > 0
GROUP BY
  artist_name,
  year
ORDER BY
  num_songs DESC,
  year DESC
```

### Criando e Executando Workflow de Tarefas do Pipeline

Acessar o menu de Workflows > Create Job > Montar a sequência de pipelines > Disparar Run Now

Obs: Para o cluster podemos utilizar o que já foi criado ou utilizar o recomendado pela Databricks

### Agendamento do job pipeline

Acesar o menu Workflows > Acessar o Workflow criado > Schedules & Trigger


### Finalizando

Basta deletar o cluster de máquinas na plataforma Databricks (Compute + Schedules) e também acessar a plataforma AWS para deletar o Stack do CloudFormation

Obs: Todos os notebooks (Pipelines) criados permanecerão na memória do Databricks









