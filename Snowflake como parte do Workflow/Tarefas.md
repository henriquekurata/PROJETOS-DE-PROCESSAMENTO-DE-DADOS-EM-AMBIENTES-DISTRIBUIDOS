# ***Usando Snowflake como parte do Workflow***


## Ferramentas:
Snowflake

## Passos: 
Já listados junto aos comandos.


## Comandos:

### Criação da conta no Snowflake

Escolher o provedor de nuvem > AWS > Criar conta


### Amostra dos dados utilizados

#Dados 

Idade,Salario,Genero,Escolaridade,Score_Exame_Psicotecnico

58,30404.959338675162,Masculino,MÃ©dio,56.0

48,20886.50240089703,Feminino,MÃ©dio,

34,15959.388748213623,Masculino,MÃ©dio,57.0

62,28534.995326705834,Outro,Superior,68.0

### Carga dos dados

Acessar Data > database > + Database > Acessar o banco de dados > + Schema > Acessar Schema > Create > Table > From file > Arrastar a tabela da máquina local para o Snowflake > Load


### Dashboards

Acessar projects > Dashboards > + Dashboard > New Tile > From SQL Worksheet 

### Consultas

Query SQL para fazer a média:
```
select escolaridade, round(avg(salario), 2) as media_salario
from dsa_db.schema1.funcionarios
group by escolaridade
```

Obs: Nessa seção é possível visualizar os dados através do SQL e também fazer visualização de gráficos


### Criando aplicação de dados com Streamlit

Acessar projects > Streamlit > + Streamlit App > Criar Streamlit

Obs: Caso seja necessário, é preciso dar privilégios de acesso ao banco de dados



### Caso de Uso - Pré-Processamento Para Machine Learning

#Substituindo valor nulo pela média
```
SELECT 
    idade, 
    CASE
        WHEN salario IS NULL THEN media_salario
        ELSE salario
    END AS salario,
    genero, 
    escolaridade,
    CASE
        WHEN score_exame_psicotecnico IS NULL THEN media_score
        ELSE score_exame_psicotecnico
    END AS score
FROM 
    dsa_db.schema1.funcionarios,
    (SELECT ROUND(AVG(salario), 2) AS media_salario, ROUND(AVG(score_exame_psicotecnico), 2) AS media_score FROM dsa_db.schema1.funcionarios) as subquery

```