# 🚀 ***Ambiente para análise de dados com Google Big Query e linguagem Python***

## 📖 **Descrição do Projeto:**
Este projeto consiste na criação de um ambiente de análise de dados utilizando Google BigQuery como data warehouse e Python para consultas e análise de dados. O pipeline de dados é construído com BigQuery, onde são criadas as tabelas, executadas queries SQL e gerados gráficos com Looker Studio e Google Colab. Além disso, o projeto conta com a execução de scripts Python para extrair, processar e analisar dados no BigQuery, incluindo a criação de um pipeline para detecção de anomalias nos dados.



## 🛠️ Ferramentas Utilizadas:
- **Google BigQuery**: Data warehouse na nuvem que permite a execução de consultas SQL em grandes volumes de dados de forma rápida e escalável. Utilizado para armazenar e consultar dados no projeto.
- **Python**: Linguagem de programação usada para realizar análises de dados e executar scripts para consultas e processamento dos dados no BigQuery, além de criar pipelines e detectar anomalias.


## 📋 **Descrição do Processo**
1. **Criação do Projeto no Google BigQuery**:
   - Acesse a visão geral da Cloud e crie um novo projeto.
2. **Criação das Tabelas do Data Warehouse**:
   - Acesse BigQuery, escolha o projeto, e crie conjuntos de dados e tabelas conforme necessário.
3. **Execução de Consultas SQL**:
   - Execute consultas SQL nas tabelas para análises, como calcular médias e gerar relatórios.
4. **Dados**:
   - **Dimensão Cliente**: Informações sobre clientes.
   - **Dimensão Localidade**: Informações sobre lojas e locais.
   - **Dimensão Produto**: Informações sobre produtos.
   - **Dimensão Tempo**: Informações sobre datas e horários.
   - **Fato**: Dados de vendas.
5. **Criação de Gráficos**:
   - Utilize Looker Studio e Google Colab para explorar e visualizar os dados com gráficos e dashboards.
6. **Scripts Python**:
   - **Job1.py**: Executa consultas SQL no BigQuery e imprime resultados.
   - **Job2.py**: Detecta anomalias nos dados de vendas usando técnicas estatísticas.


## 💻 **Comandos:** 

### Criando o projeto do DW no GBQ

#### Acessar:

Visão geral da Cloud > Projetos > Criar projeto

---

### Criando as tabelas do DW

#### Acessar:
 
 BigQuery > Explorer > Escolher o projeto > Criar conjunto de dados (tabelas, como se fossem Data Marts) > Selecionar Conjunto de dados > Criar yabela de = "Fazer Upload" > Selecionar arquivo > Formato do arquivo > Nome da tabela > Tipo de tabela = "Tabela nativa" > Esquema = "Detectar automaticamente" > Criar tabela 

---

### Dados (Amostra)

#### DimensaoCliente

```
Cliente_ID,Nome,Email,Nivel_Educacional

0,Henrique FogaÃ§a,maite55@example.com,GraduaÃ§Ã£o

1,Milena Barros,tcardoso@example.com,Ensino MÃ©dio

2,Vitor Nascimento,matheus38@example.org,PÃ³s-GraduaÃ§Ã£o

3,Maria Sophia Nogueira,catarinamartins@example.net,Ensino 
MÃ©dio

4,Emanuel Duarte,isisramos@example.net,PÃ³s-GraduaÃ§Ã£o
```

#### DimensaoLocalidade

```
Localidade_ID,Loja,Estado,Pais

0,Loja 4,ParaÃ­ba,Brasil

1,Loja 4,PiauÃ­,Brasil

2,Loja 2,Tocantins,Brasil

3,Loja 4,Santa Catarina,Brasil

4,Loja 3,ParÃ¡,Brasil
```

#### DimensaoProduto

```
Produto_ID,Nome,Preco

0,Lavadora,2513.53

1,Game Console,1535.74

2,TV 4K,2014.7

3,Lavadora,1789.14

4,Refrigerador,2204.26
```

#### DimensaoTempo

```
Tempo_ID,Data,Hora

0,2023-04-29,07:46:13

1,2023-11-08,08:47:24

2,2023-04-26,11:11:02

3,2023-03-11,11:55:10

4,2023-06-21,08:14:07
```

#### Fato

```
Venda_ID,Cliente_ID,Produto_ID,Tempo_ID,Localidade_ID,Quantidade,Total

0,39,40,98,61,2,5937.62

1,38,55,31,62,2,3466.5

2,88,17,63,41,6,11278.86

3,36,4,78,30,3,6612.78

4,55,60,96,41,5,14593.85
```

---


### Executando consultas SQL no DW

Selecione a tabela > Consulta > Em uma nova guia > Executar query SQL > Exemplo:

```
SELECT Loja, round(avg(total), 2) as Media_Venda  
FROM `dwprojeto3-405116.databasep3.FATO_VENDA` as A, `databasep3.DIMENSAO_LOCALIDADE` as B
WHERE A.Localidade_ID = B.Localidade_ID
GROUP BY Loja
ORDER BY Loja
```
---

### Criando gráficos com Looker Studio e Google Colab no DW

#### Após executar a query SQL: 

Selecionar Explorar dados > Explorar com as planilhas ou Explorar com o Looker Studio ou Explorar com Google Colab (Fornece alguns scripts Python, porém também dá para realizar tarefas com o próprio programa)

#Para executar o google Colab na máquina local é necessário criar uma conta de serviço, criar chave e instalar Python:

Acessar menu > AIM e administrador > Conta de serviço > PErfil necessário (Papel = "Editor") > Criar conta de serviço 

Acessar Chaves (IAM e adminstrador > Contas de servço) > Parte superior > Adicionar chave > Criar nova chave > Json

Instalar interpretador da linguagem Python na máquina local > Anaconda Python 

Abrir o CMD e acessar pelo terminal o local do arquivo Job1.py > pip install google.cloud > pip install google-cloud-bigquery > python job1.py (À seguir)

---

### Jo1.py

```python
import os
from google.cloud import bigquery

# Define o caminho para o arquivo de credenciais (coloque o caminho da chave JSON no seu computador criada no item anterior)
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = "/Users/kurata2/Downloads/dwprojeto3-405116-a38468e68d41.json" 

print("Service Account KEY:", os.environ['GOOGLE_APPLICATION_CREDENTIALS'])

# Cria o cliente
client = bigquery.Client()

# Define a query
query = """
   	SELECT LOJA, avg(Total) as Media_Venda
	FROM `databasep3.DIMENSAO_LOCALIDADE` as A, `databasep3.FATO_VENDA` as B
	WHERE A.Localidade_ID = B.Localidade_ID
	GROUP BY Loja
	ORDER BY Loja
"""

# Executa a query no DW
query_job = client.query(query)

print("Dados Extraídos do DW:\n")

# Loop pela query para extrair e imprimir os dados
for row in query_job:
    print("loja={}, media={}".format(row[0], row["Media_Venda"]))
```
---

### Pipeline de Detecção de Anomalias

```python
# Imports
import os
import numpy as np
from google.cloud import bigquery

print("\nPipeline de Detecção de Anomalias em DW!")

# Define o caminho para o arquivo de credenciais (coloque o caminho da chave no seu computador)
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = "/Users/kurata2/Downloads/dwprojeto3-405116-a38468e68d41.json"

print("\nService Account KEY:", os.environ['GOOGLE_APPLICATION_CREDENTIALS'])

# Cria o cliente
client = bigquery.Client()

# Define a query
query = """
   	SELECT Nome, sum(Quantidade) as Total_Quantidade
    FROM `databasep3.DIMENSAO_PRODUTO` as A, `databasep3.FATO_VENDA` as C
    WHERE A.Produto_ID = C.Produto_ID
    GROUP BY Nome
    ORDER BY Nome
"""
```
---


### Executa a query anterior no DW para identificação de anomalias

#### Job2.py

```python
query_job = client.query(query)

# Inicializa lista e dicionário
total_unidades = []
dict_prod_unidades = {}

print("\nTotal de Unidades Vendidas Por Produto:\n")

# Loop pela query para extrair os dados
for row in query_job:
    print("produto={}, total_unidades_vendidas={}".format(row[0], row["Total_Quantidade"]))
    total_unidades.append(row["Total_Quantidade"])
    dict_prod_unidades[row[0]] = row["Total_Quantidade"]

# Calculando Q1, Q3 e IQR
Q1 = np.percentile(total_unidades, 25)
Q3 = np.percentile(total_unidades, 75)
IQR = Q3 - Q1

# Definindo os limites para as anomalias
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Identificando as anomalias
print("\nIdentificando Anomalias...")
anomalias = [unidades for unidades in total_unidades if unidades < lower_bound or unidades > upper_bound]

print('\nTotal de Unidades Vendidas Que Podem Indicar Uma Anomalia: ', anomalias)

# Loop
for chave, valor in dict_prod_unidades.items():
    for elemento in anomalias:
        if valor == elemento:
            print("Produto(s) com Total de Unidades Vendidas Representando Uma Anomalia:", chave)

print("\nPipeline Concluído com Sucesso!\n")

```


---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- 🐱 [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)
