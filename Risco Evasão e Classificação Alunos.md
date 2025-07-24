#### Importando Bibliotecas e Carregando os Dados

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import (
    classification_report,
    confusion_matrix,
    roc_auc_score,
    roc_curve
)
import matplotlib.pyplot as plt
import seaborn as sns


#Carregando os dados já tratados em arquivos csv (poderia ser do DW sem problemas, é literalmente a mesma coisa)

df = pd.read_csv("dados_tratados/fato_final_aluno.csv")
```

#### Criando o Risco de Evasão

```python
Criando a risco de evasão (é um tipo binário)

df["risco_evasao"] = np.where(
    (df["semanas_presentes"] <= 4) |
    (df["media_nota"] < 3) |
    (df["total_avaliacoes"] == 0),
    1, 0
)
```

#### Selecionando as Features

```python
#Selecionando as features e tratando os valores ausentes pra não dar b.o mais tarde

features = [
    "idade", "semanas_presentes", "media_nota", "total_feedbacks",
    "peso_inicial", "peso_final", "perda_peso",
    "ganho_forca", "total_avaliacoes"
]

df_modelo = df[features + ["risco_evasao"]].copy()
df_modelo = df_modelo.fillna(0)  # substitui NaN por 0
```

#### Treinando o Modelo

```python
#Separando os dados em treino e teste (padrão)

X = df_modelo.drop("risco_evasao", axis=1)
y = df_modelo["risco_evasao"]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=42)


#Treinando o modelo de classificação (a mágica acontecendo rsrs)

modelo = RandomForestClassifier(n_estimators=100, random_state=42)
modelo.fit(X_train, y_train)
```

#### Avaliando o Desempenho do Modelo

```python
#Avaliando o desempenho do meu modelo

y_pred = modelo.predict(X_test)
y_proba = modelo.predict_proba(X_test)[:, 1]

print("Classificação:")
print(classification_report(y_test, y_pred))
```

<img width="459" height="183" alt="image" src="https://github.com/user-attachments/assets/fd22c263-a69b-425e-b28f-1f422dec4199" />

#### Matriz de Confusão

```python
# Matriz de Confusão
cm = confusion_matrix(y_test, y_pred)
sns.heatmap(cm, annot=True, fmt="d", cmap="Blues")
plt.title("Matriz de Confusão")
plt.xlabel("Predito")
plt.ylabel("Real")
plt.show()
```

<img width="539" height="455" alt="image" src="https://github.com/user-attachments/assets/b2ff081f-1715-402c-a343-b83db1b24593" />

#### Curva Roc

```python
# Crva ROC
fpr, tpr, _ = roc_curve(y_test, y_proba)
roc_auc = roc_auc_score(y_test, y_proba)
plt.figure()
plt.plot(fpr, tpr, label=f"AUC = {roc_auc:.2f}")
plt.plot([0, 1], [0, 1], "--", color="gray")
plt.title("Curva ROC")
plt.xlabel("FPR")
plt.ylabel("TPR")
plt.legend()
plt.show()
```

<img width="567" height="455" alt="image" src="https://github.com/user-attachments/assets/6ab089cd-7fe8-439e-893a-783832eb56a9" />

#### Alunos com Risco de Evasão

```python
#Contagem de alunos classificados como risco de uma evasão
print("Distribuição de risco:")
print(df_modelo['risco_evasao'].value_counts())

#Porcentagem
print("\nPorcentagem:")
print(df_modelo['risco_evasao'].value_counts(normalize=True) * 100)
```

<img width="295" height="218" alt="image" src="https://github.com/user-attachments/assets/2383fa06-0ffb-4b8c-9a01-625d51158f3b" />

#### Caracteristícas de Alunos com Risco de Evasão

```python
#Características dos alunos em risco de evasão
alunos_risco = df[df['risco_evasao'] == 1]
print("\nCaracterísticas médias dos alunos em risco:")
print(alunos_risco[features].mean())
```

<img width="362" height="220" alt="image" src="https://github.com/user-attachments/assets/2f55dd58-e655-43b9-8238-d8875015e162" />


#### Comparação Entre Alunos com Risco e Sem Risco de Evasão

```python
#Comparação com alunos sem risco de evasão
alunos_seguros = df[df['risco_evasao'] == 0]
print("\nComparação com alunos sem risco:")
print(alunos_seguros[features].mean())
```

<img width="283" height="219" alt="image" src="https://github.com/user-attachments/assets/7cdc3e72-e3bd-4525-bcd0-86624ea16c65" />

#### Analisando as Probabilidades de Risco

```python
#Adicionando a probabilidade de evasão ao DF original
df['probabilidade_evasao'] = modelo.predict_proba(df_modelo[features])[:,1]

#Analisando a distribuição das probabilidades
plt.figure(figsize=(10, 6))
sns.histplot(data=df, x='probabilidade_evasao', hue='risco_evasao', bins=30, kde=True)
plt.title('Distribuição das Probabilidades de Evasão')
plt.show()
```

<img width="859" height="547" alt="image" src="https://github.com/user-attachments/assets/905c1dd5-e627-4d51-acec-fd3b1cebaf59" />

#### Classificação de Risco de Evasão Por Nível

```python
#Classificação 
df['nivel_risco'] = pd.cut(df['probabilidade_evasao'],
                          bins=[0, 0.3, 0.7, 1],
                          labels=['Baixo', 'Médio', 'Alto'])

#Visualizando a distribuição
print("\nDistribuição por nível de risco:")
print(df['nivel_risco'].value_counts())

#Exportar uma lista de alunos com prioridades e risco de evasão para um arquivo CSV
df[['nome', 'probabilidade_evasao', 'nivel_risco']].sort_values(
    'probabilidade_evasao', ascending=False).to_csv('alunos_priorizados.csv', index=False)
```

<img width="276" height="147" alt="image" src="https://github.com/user-attachments/assets/09fec25e-0b03-45de-98d9-7e1b5fedc870" />
