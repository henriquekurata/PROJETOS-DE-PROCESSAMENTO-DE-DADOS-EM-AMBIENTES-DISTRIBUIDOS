# ***Processamento distribuído na nuvem com Amazon EMR no EC2 com PySpark***

## Ferramentas: 

Amazon EMR

## Passos:

Já listados junto aos comandos


## Comandos:

### Criação do cluster EMR com Spark
Acessar AWS > Amazon EMR > Criar Cluster > Versão Amazon EMR com Spark > Não usar catálogo de dados do AWS Glue > Sistema operacional Linux > Aplicar as atualizações mais recentes > Grupo de instâncias (Escolher as máquinas - Primário semelhante ao master, núcleo semelhante ao worker e tarefa para adicionar potência ao processamento dos dados) >
Definir o tamanho do cluster manualmente > Configuração de rede (VPC e Subnet - onde estarão os endereços IP das máquinas - Deixar a padrão sugerida) > Grupos de segurança EC2 Firewall (Deixar o padrão sugerido) > Término do Cluster (Encerrar automaticamente - Recomendado) > Ações de bootstrap (personalização de configurações) > Logs do cluster (Bucket S3 - Deixar sugerido) > 
Configurações de segurança e par de chaves do EC2 (Criar as chaves para acesso remoto ao cluster: Criar par de chaves > Tipo RSA e .ppk (para Windows) > Fazer o download das chaves > Criar chaves > Navegar > Escolher as chaves criadas)> Perfil e serviço do Amazon EMR > Escolha um perfil de serviço (Escolher o sugerido para vpc e subnet, já para o grupo de segurança utilizar o default) > Perfil de instância do EC2 >
Escolha um perfil de instância > Bucket ou prefixos específicos do S3 nessa conta com acesso de leitura e gravação > Criar CLuster

### Acessando o cluster remotamente

Acessar o Amazon EMR > Cluster > Resumo > Conectar ao nó primário usando SSH > 
Baixar o Putty de acordo com a arquitetura do computador > Iniciar o Putty > Colocar o endereço do cluster > Configurar as chaver ppk > Abrir

Obs: Caso haja problemas de acesso como "timeout" basta acessar o cluster > propriedades > Rede e segurança > Grupos de segurança do EC2 (firewall) > Nó primário > 
Editar regras de entrada (Criar regra: SSH > TCP 22 > Qualquer 0.0.0.0./0)


Acessar cluster EMR > Aplicativos > UIs de aplicativo no nó primário > Para acessar essas portas é necessário liberar o acesso > 
Editar regras de entrada > (Todos os TCPs > Presonalizado > Qualquer > 0.0.0.0./0) > Salvar regras


### Executandos os pipelines no cluster:
### Tarefa 1 - Extrair Dados de Texto 

#Usaremos a url no formato abaixo (exemplo):
#https://www.gutenberg.org/files/136/136.txt

#Conecte via SSH no Cluster EMR.

#Crie uma pasta no servidor:
mkdir dsa-dados-entrada-local

#Entre na pasta:
cd dsa-dados-entrada-local

#Crie um script sh
vi tarefa1.sh

#Coloque o conteúdo abaixo no script:

```
#!/bin/bash 
for i in {1340..1400} 
do 
    wget "http://www.gutenberg.org/files/$i/$i.txt" 
done
```

#Altere a permissão do arquivo para torná-lo um executável:
chmod +x tarefa1.sh

#Execute a tarefa 1:
./tarefa1.sh




### Tarefa 2 - Mover os Dados de Texto Para o Sistema de Arquivos Distribuído

#Acesse a pasta home do usuário no Cluster EMR

cd ~

#Verifica se o HDFS está disponível e então cria uma pasta no 

HDFS:

hdfs dfs -ls /

hdfs dfs -mkdir /user/hadoop/dsa-dados-entrada-dfs

hdfs dfs -ls /user/hadoop

#Copie os arquivos de texto no formato txt do sistema de arquivos local para o sistema de arquivos distribuído:

hdfs dfs -put dsa-dados-entrada-local/*.txt dsa-dados-entrada-dfs

# Verifique se os arquivos estão agora no ambiente distribuído:

hdfs dfs -ls /

hdfs dfs -ls /user

hdfs dfs -ls /user/hadoop

hdfs dfs -ls /user/hadoop/dsa-dados-entrada-dfs

hdfs dfs -ls /user/hadoop/dsa-dados-entrada-dfs/1340.txt



### Tarefa 3 - Criar e Executar o Pipeline

#Acesse a pasta home do usuário no Cluster EMR

cd ~

#Crie o arquivo para o script do Pipeline:

vi projeto6.py

#Coloque o conteúdo abaixo no script:

```
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

#Execute o script e submete o job para o Spark:
spark-submit projeto6.py




### Tarefa 4 - Manipular os Dados de Texto Após o Processamento

#Verfique se o resultado foi gravado no HDFS:

hdfs dfs -ls /user/hadoop/dsa-dados-saida-dfs/*

#Crie uma pasta para gravar os dados de saída no sistema de arquivos local:

mkdir dsa-dados-saida-local

#Entre na pasta:

cd dsa-dados-saida-local

#Copie os arquivos de saída do sistema de arquivos distribuído para o sistema de arquivos local:

hdfs dfs -get /user/hadoop/dsa-dados-saida-dfs/*



### Tarefa 5 - Combinar os Arquivos de Saída e Obter o Resultado do Pipeline no Cluster EMR

#Acesse a pasta com os dados de saída no sistema local no Cluster EMR:

cd dsa-dados-saida-local

#Crie o arquivo sh:

vi combina_json.sh

#Coloque no arquivo o conteúdo abaixo para combinar os arquivos JSON e gerar um único arquivo de saída:
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

#Acesse o console do S3 e faça o download do arquivo para a máquina local.


