#### Imports e Criação das Recomendações

```python
import pandas as pd

#Carregando od dados tratados (poderia ser direto do DW, seria a mesma coisa)
df = pd.read_csv('dados_modelo/cluster_alunos.csv')

#Criando as recomendações e salvando em regras
regras = {
    0: "Intensifique o treino com metas mensais personalizadas.",
    1: "Ofereça um período gratuito com personal trainer para aumentar o engajamento.",
    2: "Reavalie o plano de treino e reforce acompanhamento nutricional.",
    3: "Envie boas-vindas e lembretes das aulas semanais para manter o ritmo.",
    4: "Agende uma conversa com o instrutor para entender os feedbacks negativos."
}

#Aplicando as recomendações
df['recomendacao'] = df['cluster'].map(regras)

#Salvando em um arquivo CSV as recomendações
df.to_csv('dados_recomendacao/alunos_recomendacoes.csv', index=False)
print("Arquivo salvo.")
```

#### Inserindo os Dados no DW - Criando uma Tabela Fato

```python
import pyodbc
from datetime import datetime

#Conectando com o DW
conn = pyodbc.connect(
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'Trusted_Connection=yes;'
    r'DATABASE=AcademiaDW;'  
)
cursor = conn.cursor()

#Inserindo os dados com a data atual
for _, row in df.iterrows():
    cursor.execute("""
        INSERT INTO FatoRecomendacao (id_aluno, data_recomendacao, cluster, recomendacao)
        VALUES (?, ?, ?, ?)
    """, int(row['id_usuario']), datetime.today().date(), int(row['cluster']), row['recomendacao'])

conn.commit()

print("Dados Inseridos no BD.")
```

#### Realizando uma Consulta de Recomendação pelos Clusters

```python
import warnings
warnings.filterwarnings("ignore")

query_recomendacao_cluster = """
SELECT cluster, recomendacao, COUNT(*) as qtd
FROM FatoRecomendacao
GROUP BY cluster, recomendacao
ORDER BY qtd DESC;
"""

df_recomendacao_cluster = pd.read_sql(query_recomendacao_cluster, conn)

print("\nRecomendação Mais Comum por Cluster:")
print(df_recomendacao_cluster.to_string(index=False))
```

<img width="763" height="155" alt="image" src="https://github.com/user-attachments/assets/73977ca5-9467-402a-ae02-bb3f88606592" />

#### Última Recomendação

```python
query_ultima_recomendacao = """
SELECT id_aluno, MAX(data_recomendacao) as ultima_data, recomendacao
FROM FatoRecomendacao
GROUP BY id_aluno, recomendacao;
"""

df_ultima_recomendacao = pd.read_sql(query_ultima_recomendacao, conn)

#Convertendo os dados para datetime 
df_ultima_recomendacao['ultima_data'] = pd.to_datetime(df_ultima_recomendacao['ultima_data'])

#Formatando a data
df_ultima_recomendacao['ultima_data'] = df_ultima_recomendacao['ultima_data'].dt.strftime('%d/%m/%Y')

print("\nÚltima Recomendação por Aluno:")
print(df_ultima_recomendacao.to_string(index=False))
```

<img width="788" height="563" alt="image" src="https://github.com/user-attachments/assets/3a3b5bca-52bc-4008-ab23-d7831d3c65ec" />


