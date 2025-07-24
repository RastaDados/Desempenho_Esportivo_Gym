#### Imports e Conexão ao MongoDB (DataLake)

```python
import os
import pandas as pd
import numpy as np
from pymongo import MongoClient
from datetime import datetime


#Criando pasta para armazenar os dados tratados
os.makedirs("dados_tratados", exist_ok=True)


#Conectando ao MongoDB (DataLake)
client = MongoClient("mongodb://localhost:27017/")
db = client["academia_raw"]
```

#### Função para Carregar os Dados para um DF

```python
#Criando uma função para carregar os de cada coleção para o DF
def carregar_colecao(nome):
    return pd.DataFrame(list(db[nome].find()))

#Carregando os dados das coleções
df_usuarios = carregar_colecao("usuarios")
df_presencas = carregar_colecao("presencas")
df_feedbacks = carregar_colecao("feedbacks")
df_treinos = carregar_colecao("treinos")
df_avaliacoes = carregar_colecao("avaliacoes_fisicas")
df_tempo = carregar_colecao("dim_tempo")
df_dim_treino = carregar_colecao("dim_treino")
df_dim_instrutor = carregar_colecao("dim_instrutor")
```

#### Criação de Presenças para a Tabela Fato Frequência

```python

#Criando as presenças para inserir na Fato Frequência
df_presencas['data_entrada'] = pd.to_datetime(df_presencas['data_entrada'])
df_presencas['ano'] = df_presencas['data_entrada'].dt.year
df_presencas['semana'] = df_presencas['data_entrada'].dt.isocalendar().week
```

#### Criação da Frequência Semanal

```python
#Criando a frequência semanal
df_frequencia = df_presencas.groupby(['id_usuario', 'ano', 'semana']).agg(
    total_presencas=('id_presenca', 'count'),
    ultima_presenca=('data_entrada', 'max')
).reset_index()
```

#### Criando a Aderência de Frequência

```python
#Crianao o DF Aderência: quantas semanas o aluno foi na academia
df_aderencia = df_frequencia.groupby('id_usuario').agg(
    semanas_presentes=('semana', 'count'),
    ultima_frequencia=('ultima_presenca', 'max')
).reset_index()
```

#### Criação de Feedback para a Tabela Fato Satisfação

```python
#Criando o feedback para inserir na Fato Satisfação
df_feedbacks['data'] = pd.to_datetime(df_feedbacks['data'])

df_feedbacks_agg = df_feedbacks.groupby('id_usuario').agg(
    media_nota=('avaliacao', 'mean'),
    total_feedbacks=('avaliacao', 'count'),
    ultima_avaliacao=('data', 'max')
).reset_index()
```

#### Inserindo Avaliações Físicas

```python
#Avaliações Físicas para inserir no Progresso
df_avaliacoes['data'] = pd.to_datetime(df_avaliacoes['data'])
df_avaliacoes = df_avaliacoes.sort_values(['id_usuario', 'data'])
```

#### Cálculo de Progresso por Usuário

```python
#Calculando o progresso por usuário (aluno)
df_progresso = df_avaliacoes.groupby('id_usuario').agg(
    peso_inicial=('peso_kg', 'first'),
    peso_final=('peso_kg', 'last'),
    gordura_inicial=('gordura_percentual', 'first'),
    gordura_final=('gordura_percentual', 'last'),
    forca_inicial=('forca_maxima', 'first'),
    forca_final=('forca_maxima', 'last'),
    total_avaliacoes=('id_avaliacao', 'count')
).reset_index()

df_progresso['perda_peso'] = df_progresso['peso_inicial'] - df_progresso['peso_final']
df_progresso['ganho_forca'] = df_progresso['forca_final'] - df_progresso['forca_inicial']
```

#### Criação da Tabela Fato Final Aluno e Unindo Tudo

```python
#Criando a Fato final e unindo tudo
df_fato_final = df_usuarios.merge(df_aderencia, on='id_usuario', how='left') \
                           .merge(df_feedbacks_agg, on='id_usuario', how='left') \
                           .merge(df_progresso, on='id_usuario', how='left')
```

#### Salvando os Dados Tratados e Importandopara CSV na Pasta Criada

```python
#Salvando todos os dados tratados em CSVs separados na pasta criada
df_frequencia.to_csv("dados_tratados/fato_frequencia.csv", index=False)
df_feedbacks.to_csv("dados_tratados/fato_feedback.csv", index=False)
df_progresso.to_csv("dados_tratados/fato_avaliacao.csv", index=False)
df_fato_final.to_csv("dados_tratados/fato_final_aluno.csv", index=False)

df_usuarios.to_csv("dados_tratados/dim_usuario.csv", index=False)
df_tempo.to_csv("dados_tratados/dim_tempo.csv", index=False)
df_dim_treino.to_csv("dados_tratados/dim_treino.csv", index=False)
df_dim_instrutor.to_csv("dados_tratados/dim_instrutor.csv", index=False)

#Se tudo der certo, retorna:
print("Dados tratados e salvos.")
```

<img width="211" height="36" alt="image" src="https://github.com/user-attachments/assets/9ebd223a-0779-4908-b970-e981002f91be" />
