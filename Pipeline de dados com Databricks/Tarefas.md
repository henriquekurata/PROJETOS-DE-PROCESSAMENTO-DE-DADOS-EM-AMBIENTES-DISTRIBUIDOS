# ***Pipeline para processamento de dados com banco relacional, Airbyte e Databricks***


## Ferramentas: 

Docker, PostgreSQL, Airbyte, Databricks e AWS.

## Passos:

* Banco de dados Relacional (local-Docker-PostgreSQL) > Airbyte (Local-Docker)> Data Lakehouse (Nuvem-Databricks-AWS)

Já estão listados junto aos comandos.



## Comandos:

### Fonte de dados

Instalar o Docker de acordo com o sistema operacional da máquina local

Executar o comando para baixar a imagem e criar o container:

docker run --name db-dsa-fonte -p 5432:5432 -e POSTGRES_USER=dbadmin -e POSTGRES_PASSWORD=dbadmin123 -e POSTGRES_DB=postgresDSADB -d postgres

Container Docker = db-dsa-fonte
Banco de dados = postgresDSADB
Schema = dbadmin


### Construindo e carregando o banco de dados local como origem dos dados

#Instale o pgAdmin, crie a conexão para o banco de dados e execute as instruções SQL abaixo
https://www.pgadmin.org

#Criar schema
``` 
CREATE SCHEMA dbadmin AUTHORIZATION dbadmin;
```

#Criar tabela
```
CREATE TABLE dbadmin.tb_usuarios
(
    id_usuario integer NOT NULL,
    nome_usuario character varying(50),
    cod_cidade character varying(5),
    PRIMARY KEY (id_usuario)
);
```

```
CREATE TABLE dbadmin.tb_cidades
(
    codigo_cidade character varying(5),
    nome_cidade character varying(50),
    PRIMARY KEY (codigo_cidade)
);
```

#Carregar dados
```
INSERT INTO dbadmin.tb_cidades(codigo_cidade, nome_cidade)
VALUES ('FOR01', 'Fortaleza');

INSERT INTO dbadmin.tb_cidades(codigo_cidade, nome_cidade)
VALUES ('BLU01', 'Blumenau');

INSERT INTO dbadmin.tb_cidades(codigo_cidade, nome_cidade)
VALUES ('UBA01', 'Ubatuba');

INSERT INTO dbadmin.tb_usuarios(id_usuario, nome_usuario, cod_cidade)
VALUES (1001, 'Bob Silva', 'BLU01');

INSERT INTO dbadmin.tb_usuarios(id_usuario, nome_usuario, cod_cidade)
VALUES (1002, 'Monica Teixeira', 'BLU01');

INSERT INTO dbadmin.tb_usuarios(id_usuario, nome_usuario, cod_cidade)
VALUES (1003, 'Josenildo Farias', 'FOR01');

INSERT INTO dbadmin.tb_usuarios(id_usuario, nome_usuario, cod_cidade)
VALUES (1004, 'Maria Joy', 'UBA01');

INSERT INTO dbadmin.tb_usuarios(id_usuario, nome_usuario, cod_cidade)
VALUES (1005, 'Alex Tavares', 'FOR01');
```


#Criar chave estrangeira
```
ALTER TABLE dbadmin.tb_usuarios
    ADD CONSTRAINT "FK_CIDADE" FOREIGN KEY (cod_cidade)
    REFERENCES dbadmin.tb_cidades (codigo_cidade)
    ON UPDATE NO ACTION
    ON DELETE NO ACTION;
```



### Instalando o Airbyte localmente no docker 

#Execute os comandos abaixo para instalar o Airbyte

#Se necessário, instale o Git: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git (De acordo com o sistema operacional)

git clone https://github.com/airbytehq/airbyte.git

cd airbyte

docker-compose up



#Preparar ambiente em nuvem com Databricks

Criar conta para Databricks e AWS

No Databricks criar um bucket S3 (para armazenamento dos dados processados) para configurar a inicialização do CloudFormation (CloudFormation > Create Stack > Workspace configuration > Data bucket name)

Criar a Stack do CloudFormation com arquivo yaml da própria Databricks
Acessar Databricks > Workspace > Abrir as configurações da Worskspace > Acessar Compute > Criar Cluster de máquinas EC2 de acordo com o hardware necessário

Obs: Dentro do cluster EC2 pela Databricks existem outras configurações que podem ser acessadas: notebooks, libraries, event log, spark ui, driver logs, metrics, apps, spark cluster UI-Master



### Configuração do Airbyte

#Fonte - PostgreSQL

Configurar a fonte (PostgresLocal) com os dados do conteiner Postgres

Clicar em setup source

#Destino - Databricks

Configurar o destino com os dados do cluster criado na Databricks: 

Server Hostname/ Port / HTTP Path: Acessar Workspace > Compute > Cluster > Advanced options > JDBC/ODBC > Cópia dos detalhes de conexão para o Airbyte

Aceitar os termos de uso: Agree to the databricks JDBC

Access Token: Ícone de usuário > Settings > User > Developer > Access Tokens > Generate new token 
Database Schema: dbadmin (mesmo nome da fonte de dados)

s3 bucket name: nome bucket S3

S3 bucket path: criar uma pasta com o nome "dados" no bucket S3

S3 bucket region: Us-east-2

S3 Access Key id / S3 secret Access Key: Security credentials > Criar chave

S3 Filename: {date} (É importante fazer essa configuração para tornar possível a execução do pipeline por várias vezes)

Clicar em setup destination


#Configurando a conexão Airbyte

Selecionar sorce > selecionar destination > editar transfer (agendamento de execução) > destination namespace (mirror: levará a mesma estrutura a para o destino) > destination Stream Prefix (prefixo para diferenciar o destino) 

Full refresh/Overwrite: Apaga e grava todos os dados novamente

Incremental/Append: Adicionar apenas os dados que ainda não existem no destino (para isso é importante selecionar a chave primária dessa tabela)

Clicar em sync now (Execução do pipeline de integração de dados)


### Pipeline de Analytics com Apache Spark no Databricks com Python

Abrir Workspace :

#Listar o conteúdo no data lake:
```
dbutils.fs.ls("s3://projeto5-dsa-analytics/dados/dbadmin/") - Entre parenteses é a URI que estão as duas tabelas criadas no S3
```

#Ler a tabela e gravar em um dataframe:
```
df = spark.read.load("s3://projeto5-dsa-analytics/dados/dbadmin/p5_tb_usuarios")
display(df) 
```

#Fazendo Join das duas tabelas com SQL
```
%sql
SELECT nome_usuario, nome_cidade
FROM dbadmin.p5_tb_usuarios, dbadmin.p5_tb_cidades
WHERE dbadmin.p5_tb_cidades.codigo_cidade = dbadmin.p5_tb_usuarios.cod_cidade;
```

#Criar gráfico
```
%sql
SELECT nome_cidade, COUNT(*)
FROM dbadmin.p5_tb_usuarios, dbadmin.p5_tb_cidades
WHERE dbadmin.p5_tb_cidades.codigo_cidade = dbadmin.p5_tb_usuarios.cod_cidade
GROUP BY nome_cidade;
```

Para a criação do gráfico basta acessar: sqldf > Table > + > Configurações do gráfico > Save

#Criar Dashboard

Acessar : Tabela criada > Show in Dashboard ou add to dashboard > Configurações Dashboard

Acessar o grádio > Add to dashboard 

Assim teremos o painel com as tabelas e o gráfico em visualização única
