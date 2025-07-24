#### Destaques e Insights Obtidos Após a Análise

- Baixa frequência semanal está fortemente associada à menor perda de peso e progresso físico
- Clusters revelam perfis distintos de alunos com ações direcionadas (novatos, engajados, em risco)
- Feedbacks revelam alunos com alta frequência mas baixa satisfação, o que exige revisão do plano ou atendimento
- Correlação clara entre presença, avaliações e performance, evidenciando o valor de ações preventivas

#### Importando as Bibliotecas e Carregando os Dados

```python
#Imports
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

#Algumas Estilizações
sns.set_palette("Set2")
%matplotlib inline

#Carga dos dados já tratados
df_usuario = pd.read_csv("dados_tratados/dim_usuario.csv")
df_feedback = pd.read_csv("dados_tratados/fato_feedback.csv")
df_final = pd.read_csv("dados_tratados/fato_final_aluno.csv")
df_frequencia = pd.read_csv("dados_tratados/fato_frequencia.csv")
df_cluster = pd.read_csv("dados_modelo/cluster_alunos.csv")
df_recomendacoes = pd.read_csv("dados_recomendacao/alunos_recomendacoes.csv")
```

#### Análise Inicial dos Dados Carregados

```python
#Análise Inicial dos dados carregados no DF
def resumo_inicial(df, nome):
    print(f"\n--- {nome} ---")
    print(df.shape)
    print(df.dtypes)
    print(df.isnull().sum())

resumo_inicial(df_usuario, "dim_usuario")
resumo_inicial(df_feedback, "fato_feedback")
resumo_inicial(df_final, "fato_final_aluno")
resumo_inicial(df_frequencia, "fato_frequencia")
resumo_inicial(df_cluster, "cluster_alunos")
resumo_inicial(df_recomendacoes, "alunos_recomendacoes")
```

<img width="242" height="567" alt="image" src="https://github.com/user-attachments/assets/4f2dd8d7-3419-40cb-9011-69d6e2469ed7" />

#### Distribuição de Alunos por Gênero e Idade

```python
df_usuario['genero'].value_counts().plot(kind='bar', title='Distribuição por Gênero')
plt.show()

plt.hist(df_usuario['idade'], bins=15, edgecolor='black')
plt.title('Distribuição de Idade dos Alunos')
plt.xlabel('Idade')
plt.ylabel('Quantidade')
plt.show()
```

<img width="560" height="510" alt="image" src="https://github.com/user-attachments/assets/d05b3efb-250e-4d60-a7e1-9e65812ad36d" />

<img width="571" height="455" alt="image" src="https://github.com/user-attachments/assets/af0cb7fa-b571-4848-a75f-97e764fde5f7" />

#### Análise de Frequência na Academia

```python
df_final['status'].value_counts().plot.pie(autopct='%1.1f%%', startangle=90)
plt.title('Status dos Alunos')
plt.ylabel('')
plt.show()

sns.histplot(df_final['semanas_presentes'], bins=20)
plt.title('Distribuição de Semanas Presentes')
plt.xlabel('Semanas com Presença')
plt.show()
```

<img width="415" height="411" alt="image" src="https://github.com/user-attachments/assets/1ecf706d-6f76-45e5-ae61-41960d1d4c88" />

<img width="577" height="455" alt="image" src="https://github.com/user-attachments/assets/d804263a-8ce9-4a0d-8c69-43e6384b5369" />

#### Análise de Performance Física

```python
sns.histplot(df_final['perda_peso'], kde=True, color='blue')
plt.title('Distribuição da Perda de Peso')
plt.show()

sns.histplot(df_final['ganho_forca'], kde=True, color='green')
plt.title('Distribuição do Ganho de Força')
plt.show()
```

<img width="571" height="455" alt="image" src="https://github.com/user-attachments/assets/7a0195de-27f5-45ed-bc35-af026b57d8fb" />

<img width="571" height="455" alt="image" src="https://github.com/user-attachments/assets/adb08aee-d11a-4864-9481-683b54a9bc46" />

#### Correlação Entre Variáveis de Desempenho

```python
plt.figure(figsize=(10,6))
sns.heatmap(df_final[['semanas_presentes', 'perda_peso', 'ganho_forca', 'total_avaliacoes']].corr(), annot=True, cmap='coolwarm')
plt.title('Correlações entre Variáveis de Desempenho')
plt.show()
```

<img width="757" height="528" alt="image" src="https://github.com/user-attachments/assets/4a12717f-a1bd-44c3-8a5f-da46189201f5" />

#### Análise de Feedbacks

```python
df_feedback['avaliacao'].value_counts().sort_index().plot(kind='bar')
plt.title('Distribuição das Avaliações')
plt.xlabel('Nota')
plt.ylabel('Qtd de Feedbacks')
plt.show()
```

<img width="580" height="450" alt="image" src="https://github.com/user-attachments/assets/b349339f-65f6-45cd-a8cc-afa2e8b5691f" />

#### Análise de Clusters

```pytohn
sns.countplot(x='cluster', data=df_cluster)
plt.title('Distribuição dos Alunos por Cluster')
plt.show()

sns.scatterplot(data=df_cluster, x='pca1', y='pca2', hue='cluster')
plt.title('Projeção PCA dos Clusters')
plt.show()
```

<img width="571" height="455" alt="image" src="https://github.com/user-attachments/assets/5ac61b46-db6c-4875-94d5-ceb884595d87" />

<img width="565" height="455" alt="image" src="https://github.com/user-attachments/assets/71e59e8a-c137-4504-a437-f6a6a2fd0c7b" />

#### Recomendações Para Perfis dos Alunos

```python
recs_por_cluster = df_recomendacoes.groupby('cluster')['recomendacao'].first()
for cluster, recomendacao in recs_por_cluster.items():
    print(f"\n📌 Cluster {cluster}: {recomendacao}")
```

<img width="724" height="152" alt="image" src="https://github.com/user-attachments/assets/d36ae6d1-e84e-4e42-94b2-f5b0f5a667d5" />

#### Perda de Peso Por Status na Academia

```python
plt.figure(figsize=(8, 5))
sns.boxplot(x='status', y='perda_peso', data=df_final)
plt.title('Distribuição da Perda de Peso por Status')
plt.xlabel('Status do Aluno')
plt.ylabel('Perda de Peso (kg)')
plt.show()
```

<img width="698" height="470" alt="image" src="https://github.com/user-attachments/assets/0eb603e7-2981-4641-9dba-52d46a5af950" />
