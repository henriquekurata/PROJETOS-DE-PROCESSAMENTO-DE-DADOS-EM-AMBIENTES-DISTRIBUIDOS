# 🚀 ***Processamento distribuído na nuvem com Amazon EMR no EC2 com PySpark***

## 📖 **Descrição do Projeto:**
Este projeto demonstra o uso de um cluster Amazon EMR para processar dados distribuídos utilizando PySpark. São realizados diversos passos, desde a configuração do cluster até a execução de pipelines de processamento de texto e a manipulação de dados no HDFS.



## 🛠️ Ferramentas Utilizadas:
- Amazon EMR
- EC2
- HDFS
- PySpark


## 📋 **Descrição do Processo:**

- Criação de um cluster EMR com Spark na AWS.
- Acesso remoto ao cluster via SSH para configuração.
- Extração de dados de texto da internet.
- Transferência dos dados para o HDFS (Sistema de Arquivos Distribuído Hadoop).
- Processamento dos dados com PySpark, incluindo limpeza e manipulação de texto.
- Armazenamento dos resultados no HDFS.
- Transferência dos resultados para o sistema local e o Amazon S3.



## 💻 **Comandos:** 

### Criação do Cluster EMR com Spark:
1. Acesse **AWS > Amazon EMR > Criar Cluster**.
2. Selecione a versão do Amazon EMR com Spark.
3. Não use o catálogo de dados do AWS Glue.
4. Selecione o sistema operacional **Linux** e aplique as atualizações mais recentes.
5. Defina o grupo de instâncias (máquinas primárias, núcleos e tarefas).
6. Configure o tamanho do cluster e defina rede (VPC, Subnet) e grupos de segurança.
7. Defina ações de bootstrap, logs no S3 e as chaves do EC2 para acesso remoto.
8. Crie o Cluster.

---

### Acessando o Cluster Remotamente:
1. Acesse o **Amazon EMR > Cluster > Resumo** e conecte ao nó primário via **SSH**.
2. Configure o Putty com o endereço do cluster e as chaves **ppk**.
3. Caso ocorra erro de timeout, edite as regras de entrada do firewall para liberar portas necessárias (TCP 22 e TCPs personalizados).

Obs: Caso haja problemas de acesso como "timeout" basta acessar o cluster > propriedades > Rede e segurança > Grupos de segurança do EC2 (firewall) > Nó primário > 
Editar regras de entrada (Criar regra: SSH > TCP 22 > Qualquer 0.0.0.0./0)


Acessar cluster EMR > Aplicativos > UIs de aplicativo no nó primário > Para acessar essas portas é necessário liberar o acesso > 
Editar regras de entrada > (Todos os TCPs > Presonalizado > Qualquer > 0.0.0.0./0) > Salvar regras

---

### Executandos os pipelines no cluster:

#### **Tarefa 1 - Extrair Dados de Texto**

#Usaremos a url no formato abaixo (exemplo):
#https://www.gutenberg.org/files/136/136.txt

1. Conecte-se ao cluster via **SSH**.

2. Crie uma pasta para armazenar os dados:
    ```bash
    mkdir dsa-dados-entrada-local
    cd dsa-dados-entrada-local
    vi tarefa1.sh
    ```
3. Adicione o seguinte script ao arquivo:
    ```bash
    #!/bin/bash 
    for i in {1340..1400} 
    do 
        wget "http://www.gutenberg.org/files/$i/$i.txt" 
    done
    ```
4. Execute o script para baixar os dados:
    ```bash
    chmod +x tarefa1.sh
    ./tarefa1.sh
    ```
---

### Tarefa 2 - Mover os Dados de Texto Para o Sistema de Arquivos Distribuído

#### Acesse a pasta home do usuário no Cluster EMR

cd ~

#Verifica se o HDFS está disponível e então cria uma pasta no 

HDFS:

hdfs dfs -ls /

hdfs dfs -mkdir /user/hadoop/dsa-dados-entrada-dfs

hdfs dfs -ls /user/hadoop

#Copie os arquivos de texto no formato txt do sistema de arquivos local para o sistema de arquivos distribuído:

hdfs dfs -put dsa-dados-entrada-local/*.txt dsa-dados-entrada-dfs

#### Verifique se os arquivos estão agora no ambiente distribuído:
```
hdfs dfs -ls /

hdfs dfs -ls /user

hdfs dfs -ls /user/hadoop

hdfs dfs -ls /user/hadoop/dsa-dados-entrada-dfs

hdfs dfs -ls /user/hadoop/dsa-dados-entrada-dfs/1340.txt
```
---

### Tarefa 3 - Criar e Executar o Pipeline

#Acesse a pasta home do usuário no Cluster EMR

cd ~

#Crie o arquivo para o script do Pipeline:

vi projeto6.py

#Coloque o conteúdo abaixo no script:

```py
# Imports
import re
import string
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf
from pyspark.sql.types import ArrayType, StringType

# Cria uma sessão Spark
spark = SparkSession.builder \
    .appName("DSAProjeto6") \
    .getOrCreate()

# Define uma função para limpar o texto
def limpa_texto(text):
    
    # Cria um REGEX
    # https://docs.python.org/3/library/re.html
    regex = re.compile('[%s]' % re.escape(string.punctuation))

    # Aplica o REGEX e remove pontuação
    texto_limpo = regex.sub('', text)
    
    # Converte para minúsculas e divide o texto em palavras
    words = texto_limpo.lower().split()
    
    # Cria uma lista de stopwords (palavras que não são relevantes em PLN)
    stopwords = set(["a", "an", "and", "are", "as", "at", "be", "by", "for", "from", "has", "he", "in", "is", "it", "its", "of", "on", "that", "the", "to", "was", "were", "will", "with"])
    
    # Remove stopwords
    texto_limpo_final = [word for word in words if word not in stopwords]
    
    return texto_limpo_final

# Define uma UDF para limpar o texto
# UDFs são usados para estender as funções Python e reutilizar essas funções em vários DataFrames
udf_limpa_texto = udf(limpa_texto, ArrayType(StringType()))

# Ler os arquivos de texto da pasta
# Substitua 'hdfs://[host]:[port]/[path]' pelo caminho correto para sua pasta no HDFS
# Use o endereço do seu Cluster EMR (Só alterar "/ip-172-31-2-107" pelo endereço que está na linha de comando do acesso ao cluster referente ao nó primário
df_texto_entrada = spark.read.text("hdfs://ip-172-31-2-107.us-east-2.compute.internal:8020/user/hadoop/dsa-dados-entrada-dfs")

# Limpa o texto e divide cada frase em uma lista de palavras
df_texto_saida = df_texto_entrada.withColumn("lista_palavras", udf_limpa_texto(df_texto_entrada["value"]))

# Gravar as palavras resultantes em um arquivo de saída
# Substitua 'hdfs://[host]:[port]/[path_saida]' pelo caminho correto para sua pasta de saída no HDFS
# Use o endereço do seu Cluster EMR 
df_texto_saida.write.mode("overwrite").json("hdfs://ip-172-31-2-107.us-east-2.compute.internal:8020/user/hadoop/dsa-dados-saida-dfs")

# Parar a sessão Spark
spark.stop()

```

#### Execute o script e submete o job para o Spark:
```
spark-submit projeto6.py
```

---

### Tarefa 4 - Manipular os Dados de Texto Após o Processamento

#### Verfique se o resultado foi gravado no HDFS:

hdfs dfs -ls /user/hadoop/dsa-dados-saida-dfs/*

#### Crie uma pasta para gravar os dados de saída no sistema de arquivos local:

mkdir dsa-dados-saida-local

#Entre na pasta:

cd dsa-dados-saida-local

#### Copie os arquivos de saída do sistema de arquivos distribuído para o sistema de arquivos local:

hdfs dfs -get /user/hadoop/dsa-dados-saida-dfs/*


---

### Tarefa 5 - Combinar os Arquivos de Saída e Obter o Resultado do Pipeline no Cluster EMR

#### Acesse a pasta com os dados de saída no sistema local no Cluster EMR:

cd dsa-dados-saida-local

#Crie o arquivo sh:

vi combina_json.sh

#### Coloque no arquivo o conteúdo abaixo para combinar os arquivos JSON e gerar um único arquivo de saída:
```
#!/bin/bash
# Encontra todos os arquivos JSON e os passa para jq para combinação em um único array
find . -name "*.json" -print0 | xargs -0 jq -c '.' | jq -s '.' > dsa-resultado.json

# Altere a permissão do arquivo para torná-lo um executável:
chmod +x combina_json.sh

# Execute o script:
./combina_json.sh

# Verifique o resultado:
cat dsa-resultado.json

# Zip do arquivo:
zip dsa-resultado.zip dsa-resultado.json

# Copia o arquivo zip para o S3: Endereço "aws-logs-890582101704-us-east-2" retirado da página do Amazon S3 (URI do S3)
aws s3 cp dsa-resultado.zip s3://aws-logs-890582101704-us-east-2/elasticmapreduce/dsa-resultado.zip
```

#### Acesse o console do S3 e faça o download do arquivo para a máquina local.
   ```bash
    zip dsa-resultado.zip dsa-resultado.json
    aws s3 cp dsa-resultado.zip s3://aws-logs-890582101704-us-east-2/elasticmapreduce/dsa-resultado.zip
    ```

---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- 🐱 [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)
