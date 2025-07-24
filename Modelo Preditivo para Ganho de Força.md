#### Importando Bibliotecas e Carregando os Dados Já com Cluster

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import joblib
import numpy as np


#Carregando os dados já com cluster
df = pd.read_csv("dados_modelo/cluster_alunos.csv")
```

#### Criando, Treinando e Selecionando o Melhor Modelo

```python
#Selecionando as variáveis que serão utilizadas
features = [
    "idade", "semanas_presentes", "perda_peso",
    "total_feedbacks", "media_nota", "cluster"
]
target = "ganho_forca"

X = df[features].fillna(0)
y = df[target].fillna(0)


#Dividindo em treino e teste
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)


#Criando o modelo de regressão linear (baseline)
lr = LinearRegression()
lr.fit(X_train, y_train)
y_pred_lr = lr.predict(X_test)

#Criando o modelo random forest regressor
rf = RandomForestRegressor(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)
y_pred_rf = rf.predict(X_test)

#Avaliando o desempenho dos dois modelos
def avaliar_modelo(y_test, y_pred, nome_modelo):
    print(f"\nAvaliação do Modelo: {nome_modelo}")
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 2))
    print("RMSE:", round(np.sqrt(mean_squared_error(y_test, y_pred)), 2))
    print("R²:", round(r2_score(y_test, y_pred), 3))

avaliar_modelo(y_test, y_pred_lr, "Regressão Linear")
avaliar_modelo(y_test, y_pred_rf, "Random Forest")


#Salvando o Melhor modelo
joblib.dump(rf, "modelo_regressao_forca.pkl")
print("\nModelo Salvo.")

import joblib

#Salvo o modelo
joblib.dump(modelo, "modelo_risco_evasao.pkl")

#Se tudo der certo, vai retornar:
print("Modelo salvo com sucesso.")
```

<img width="325" height="240" alt="image" src="https://github.com/user-attachments/assets/fd56ff74-a384-49ca-b1a8-691047a6092c" />

#### Comparando Valores Reais com Previstos

```python
#Criando um DF pra comparar valores reais com valores previstos
resultados = pd.DataFrame({
    "Real": y_test,
    "Predito_RF": y_pred_rf
})

print(resultados)
```

<img width="214" height="241" alt="image" src="https://github.com/user-attachments/assets/d82f6b01-b827-46f8-8002-cc64672209c2" />

```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(8, 6))
sns.scatterplot(x="Real", y="Predito_RF", data=resultados, alpha=0.7)
plt.plot([min(resultados["Real"]), max(resultados["Real"])],
         [min(resultados["Real"]), max(resultados["Real"])], 'r--')
plt.title("Comparação entre Valores Reais e Previstos (Random Forest)")
plt.xlabel("Valores Reais")
plt.ylabel("Valores Previstos")
plt.show()
```

<img width="707" height="547" alt="image" src="https://github.com/user-attachments/assets/ce83800f-1970-4f5b-82cf-5aef35646497" />

#### Tabela de Comparação Entre os Modelos

```python
from tabulate import tabulate

metrics_table = [
    ["Regressão Linear",
     mean_absolute_error(y_test, y_pred_lr),
     np.sqrt(mean_squared_error(y_test, y_pred_lr)),
     r2_score(y_test, y_pred_lr)],
    
    ["Random Forest",
     mean_absolute_error(y_test, y_pred_rf),
     np.sqrt(mean_squared_error(y_test, y_pred_rf)),
     r2_score(y_test, y_pred_rf)]
]

print(tabulate(metrics_table, headers=["Modelo", "MAE", "RMSE", "R²"], tablefmt="pretty"))
```

<img width="656" height="119" alt="image" src="https://github.com/user-attachments/assets/2b4de7ee-c39c-409b-846d-1e4a5a33ff6c" />

#### Predição de Ganho de Força de Aluno Novo

```python
# Prevendo o ganho de força de um aluno novo
novo_aluno = pd.DataFrame([{
    "idade": 30,
    "semanas_presentes": 16,
    "perda_peso": 3.0,
    "total_feedbacks": 4,
    "media_nota": 4.2,
    "cluster": 1
}])

modelo = joblib.load("modelo_regressao_forca.pkl")
ganho_previsto = modelo.predict(novo_aluno)[0]
print(f"Ganho de força previsto: {ganho_previsto:.2f} kg")
```

<img width="279" height="33" alt="image" src="https://github.com/user-attachments/assets/b3c263ea-be2c-4771-976e-b0a6c82f8eee" />
