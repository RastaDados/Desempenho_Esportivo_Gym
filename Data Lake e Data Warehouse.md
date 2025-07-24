#### Criação e Carga do Data Lake Utilizando o MongoDB

```python
from pymongo import MongoClient
import pandas as pd
import os

#Especificando o Caminho dos dados
caminho_dados = "dados"

#Conectando ao MongoDB (Data Lake)
client = MongoClient("mongodb://localhost:27017/")
db = client["academia_raw"]

#Criando uma função para inserir os dados CSV em uma coleção
def carregar_csv_para_mongo(nome_arquivo, nome_colecao):
    caminho_arquivo = os.path.join(caminho_dados, nome_arquivo)
    df = pd.read_csv(caminho_arquivo)

    #Convertendo o DF para um dicionário
    dados = df.to_dict(orient="records")

    # nserindo os dados no MongoDB
    db[nome_colecao].drop()  # limpa a coleção caso já exista
    db[nome_colecao].insert_many(dados)

#Lista de arquivos e coleções inseridos
arquivos_colecoes = {
    "usuarios.csv": "usuarios",
    "treinos.csv": "treinos",
    "presencas.csv": "presencas",
    "feedbacks.csv": "feedbacks",
    "avaliacoes_fisicas.csv": "avaliacoes_fisicas",
    "dim_tempo.csv": "dim_tempo",
    "logs_maquinas.csv": "logs_maquinas",
    "dim_treino.csv": "dim_treino",
    "dim_instrutor.csv": "dim_instrutor"
}

#Inserindo todos os arquivos no MongoDB
for arquivo, colecao in arquivos_colecoes.items():
    carregar_csv_para_mongo(arquivo, colecao)

#Se tudo der certo, retorna:
print("Processo Concluído.")
```

#### Criação e Carga do Data Warehouse Utilizando o SQLServer

```python
import pyodbc

#Conectando ao SQL Server como o Master
conn_str = (
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'Trusted_Connection=yes;'
    r'DATABASE=master;' 
)

#Conectando com o servidor, deixei o autocommit como verdadeiro
conn = pyodbc.connect(conn_str, autocommit=True)  
cursor = conn.cursor()

#Criando o banco de dados AcademiaDW 
cursor.execute("""
    IF DB_ID('AcademiaDW') IS NULL
        BEGIN
            CREATE DATABASE AcademiaDW;
        END
""")

#Se tudo der certo vai retornar:
print("Banco de dados 'AcademiaDW' criado.")

#Fechao o curso e a conexão
cursor.close()
conn.close()

#Crio uma nova conexão com o novo banco que criei acima
conn_academia = pyodbc.connect(
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'Trusted_Connection=yes;'
    r'DATABASE=AcademiaDW;'
)

#Se tudo der certo vai retornar:
print("Conectado ao banco de dados.")
```

#### Criação das Tabelas e Colunas

```python
import pyodbc

#Conectando ao banco AcademiaDW
conn = pyodbc.connect(
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'Trusted_Connection=yes;'
    r'DATABASE=AcademiaDW;'  
)
cursor = conn.cursor()

#Criando as tabelas e as colunas
sql_tabelas = [
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='dim_usuario' AND xtype='U')
    CREATE TABLE dim_usuario (
        id_usuario INT PRIMARY KEY,
        nome VARCHAR(100),
        idade INT,
        genero VARCHAR(20),
        email VARCHAR(100),
        data_inicio DATE,
        status VARCHAR(20)
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='dim_tempo' AND xtype='U')
    CREATE TABLE dim_tempo (
        data DATE PRIMARY KEY,
        ano INT,
        mes INT,
        dia INT,
        dia_semana VARCHAR(20),
        semana_ano INT
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='dim_treino' AND xtype='U')
    CREATE TABLE dim_treino (
        id_treino INT PRIMARY KEY,
        nome_treino VARCHAR(50)
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='dim_instrutor' AND xtype='U')
    CREATE TABLE dim_instrutor (
        id_instrutor INT PRIMARY KEY,
        nome VARCHAR(100),
        especialidade VARCHAR(50),
        tempo_experiencia_anos INT
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='fato_frequencia' AND xtype='U')
    CREATE TABLE fato_frequencia (
        id_usuario INT,
        ano INT,
        semana INT,
        total_presencas INT,
        ultima_presenca DATETIME
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='fato_feedback' AND xtype='U')
    CREATE TABLE fato_feedback (
        id_feedback VARCHAR(50) PRIMARY KEY,
        id_usuario INT,
        data DATE,
        avaliacao INT,
        comentario VARCHAR(255)
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='fato_avaliacao' AND xtype='U')
    CREATE TABLE fato_avaliacao (
        id_usuario INT PRIMARY KEY,
        peso_inicial FLOAT,
        peso_final FLOAT,
        gordura_inicial FLOAT,
        gordura_final FLOAT,
        forca_inicial FLOAT,
        forca_final FLOAT,
        total_avaliacoes INT,
        perda_peso FLOAT,
        ganho_forca FLOAT
    );
    """,
    """
    IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='fato_final_aluno' AND xtype='U')
    CREATE TABLE fato_final_aluno (
        id_usuario INT PRIMARY KEY,
        nome VARCHAR(100),
        idade INT,
        genero VARCHAR(20),
        email VARCHAR(100),
        data_inicio DATE,
        status VARCHAR(20),
        semanas_presentes INT,
        ultima_frequencia DATETIME,
        media_nota FLOAT,
        total_feedbacks INT,
        ultima_avaliacao DATE,
        peso_inicial FLOAT,
        peso_final FLOAT,
        gordura_inicial FLOAT,
        gordura_final FLOAT,
        forca_inicial FLOAT,
        forca_final FLOAT,
        total_avaliacoes INT,
        perda_peso FLOAT,
        ganho_forca FLOAT
    );
    """
]

#Executando o comando acima para criação das tabelas e colunas
for comando in sql_tabelas:
    cursor.execute(comando)

conn.commit()

#Se tudo der certo, retorna:
print("Tabelas Criadas.")
```

#### Carga dos Dados no DW

```python
#Fazendo a ingestão dos dados no DW

#Tratando e informando sobre o processo e erros
for arquivo, tabela in arquivos_tabelas.items():
    print(f"Carregando: {arquivo} -> {tabela}")
    caminho_arquivo = os.path.join(caminho, arquivo)
    
    try:
        df = pd.read_csv(caminho_arquivo)
    except Exception as e:
        print(f"Erro ao ler arquivo {arquivo}: {e}")
        continue

    #Removendo a coluna _id se ela existir, normalmente o Mongo faz isso por padrão (criar ids)
    if '_id' in df.columns:
        df = df.drop(columns=['_id'])

    #Identificando as colunas de data
    date_cols = [col for col in df.columns 
                if any(keyword in col.lower() for keyword in ['data', 'hora', 'time', 'dt'])]

    #Limitando os campos de texto a 255 caracteres e tratando valores ausentes (none, nan)
    for col in df.select_dtypes(include='object').columns:
        if col not in date_cols:  
            df[col] = df[col].astype(str).str[:255]
            df[col] = df[col].replace('nan', None)  

    #Tratando os campos de data e hora
    for col in date_cols:
        try:
            df[col] = pd.to_datetime(df[col], errors='coerce')
        except Exception as e:
            print(f"Erro ao converter coluna {col} para datetime: {e}")
            df[col] = None

    #Tratando campos com dados numéricos
    for col in df.select_dtypes(include=['number']).columns:
        df[col] = pd.to_numeric(df[col], errors='coerce').round(4)

    #Substituindo NaN por None para não dar b.o (= NULL no SQLServer)
    df = df.where(pd.notnull(df), None)

    #Preparando os dados para a inserção
    colunas = ",".join(df.columns)
    placeholders = ",".join(["?" for _ in df.columns])

    try:
        #Limpar os dados da tabela antes da inserção (se fossem muitos dados não faria isso)
        cursor.execute(f"DELETE FROM {tabela};")
        conn.commit()
    except Exception as e:
        print(f"Erro ao limpar tabela {tabela}: {e}")
        continue

    #Inserindo os dados linha por linha
    for index, row in df.iterrows():
        try:
            #Convertendo a linha para tupla, e tratando mais uma vez o None porque deu B.O
            values = tuple(None if pd.isna(x) else x for x in row)
            cursor.execute(f"INSERT INTO {tabela} ({colunas}) VALUES ({placeholders})", values)
        except Exception as e:
            print(f"\nErro na linha {index} ao inserir na tabela {tabela}: {e}")
            print("Continua dando B.O :(")
            for col, val in zip(df.columns, row):
                print(f"{col}: {val} (Tipo: {type(val)})")
            print("\n")
            conn.rollback()
            break
    else:
        #Commita e se tudo der certo retorna a mensagem abaixo
        conn.commit()
        print(f"Dados carregados com sucesso na tabela {tabela}")

#Se tudo der certo retorna:
conn.commit()
print("Processo concluído!")
```


