# 🚀 ***Usando Snowflake como parte do Workflow***

## 📖 **Descrição do Projeto:**
Este projeto demonstra o uso do Snowflake como parte de um workflow de análise de dados, incluindo carga de dados, consultas SQL, visualização em dashboards e pré-processamento para machine learning.


## 🛠️ Ferramentas Utilizadas:
- Snowflake
- Streamlit (para visualização)


## 📋 **Descrição do Processo:**
- Criação de conta no Snowflake e configuração do banco de dados.
- Carga de dados via Snowflake a partir de arquivos locais.
- Criação de dashboards no Snowflake para visualização dos dados.
- Consultas SQL para análise de dados, como cálculo de média salarial por escolaridade.
- Desenvolvimento de uma aplicação de dados com Streamlit.
- Pré-processamento de dados para machine learning, incluindo tratamento de valores nulos.


## 💻 **Comandos:** 

### Criação da conta no Snowflake
1. Escolher o provedor de nuvem > AWS.
2. Criar conta no Snowflake.

---

### Amostra dos dados utilizados

#### Dados 

Idade,Salario,Genero,Escolaridade,Score_Exame_Psicotecnico

58,30404.959338675162,Masculino,MÃ©dio,56.0

48,20886.50240089703,Feminino,MÃ©dio,

34,15959.388748213623,Masculino,MÃ©dio,57.0

62,28534.995326705834,Outro,Superior,68.0

---

### Carga dos dados
1. Acessar **Data** > **Database** > **+ Database**.
2. Criar o banco de dados e o esquema.
3. Arrastar os arquivos da máquina local para o Snowflake e carregar a tabela.

---

### Dashboards
1. Acessar **Projects** > **Dashboards** > **+ Dashboard**.
2. Criar um novo painel com **New Tile** > **From SQL Worksheet**.

---

### Consultas

#### Exemplo de consulta SQL para calcular a média salarial por escolaridade:

```sql
select escolaridade, round(avg(salario), 2) as media_salario
from dsa_db.schema1.funcionarios
group by escolaridade
```

Obs: Nessa seção é possível visualizar os dados através do SQL e também fazer visualização de gráficos

---

### Criando aplicação de dados com Streamlit

Acessar projects > Streamlit > + Streamlit App > Criar Streamlit

Obs: Caso seja necessário, é preciso dar privilégios de acesso ao banco de dados


---

### Caso de Uso - Pré-Processamento Para Machine Learning

#### Exemplo de substituição de valores nulos pela média:

```sql
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

---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- 🐱 [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)
