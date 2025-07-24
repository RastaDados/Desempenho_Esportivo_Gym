#### Imports e Conexão ao DW

```python
import pyodbc
import pandas as pd
import os
from datetime import datetime
import warnings
warnings.filterwarnings("ignore")

#Conectando ao SQL Server localmente (via autenticação do Windows)
conn = pyodbc.connect(
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'DATABASE=AcademiaDW;'
    r'Trusted_Connection=yes;'
)

#Criando a pasta dados_alertas 
os.makedirs("dados_alertas", exist_ok=True)
```

#### Alunos com Ausência Maior que 14 Dias

```python
#Alunos com ausência maior que 14 dias
query_ausencia = """
SELECT id_usuario, nome, ultima_frequencia,
       DATEDIFF(DAY, ultima_frequencia, GETDATE()) AS dias_sem_comparecimento
FROM fato_final_aluno
WHERE DATEDIFF(DAY, ultima_frequencia, GETDATE()) > 14;
"""

df_ausencia = pd.read_sql(query_ausencia, conn)
df_ausencia["tipo_alerta"] = "Ausência Maior Que 14 dias"
```

#### Alunos com Feedback Médio Abaixo de 3

```python
#Alunos com feedback médio abaixo de 3
query_feedback = """
SELECT id_usuario, nome, media_nota
FROM fato_final_aluno
WHERE media_nota < 3;
"""

df_feedback = pd.read_sql(query_feedback, conn)
df_feedback["tipo_alerta"] = "Feedback Menor Que 3"
```

#### Alunos sem Avaliação a Mais de 180 dias

```python
#Alunos que estão sem avaliação há mais de 180 dias
query_avaliacao = """
SELECT id_usuario, nome, MAX(ultima_avaliacao) AS ultima_avaliacao
FROM fato_final_aluno
GROUP BY id_usuario, nome
HAVING DATEDIFF(DAY, MAX(ultima_avaliacao), GETDATE()) > 180;
"""

df_avaliacao = pd.read_sql(query_avaliacao, conn)
df_avaliacao["tipo_alerta"] = "Avaliação física Maior Que 6 meses"
```

#### Unindo os Alertas e Exportando o Relatório para um CSV

```python
#Juntar todos os alertas criados
df_alertas = pd.concat([
    df_ausencia[["id_usuario", "nome", "tipo_alerta"]],
    df_feedback[["id_usuario", "nome", "tipo_alerta"]],
    df_avaliacao[["id_usuario", "nome", "tipo_alerta"]]
]).drop_duplicates()


#Exportar como relatório
hoje = datetime.today().strftime('%Y%m%d')
caminho_alerta = f"dados_alertas/alertas_{hoje}.csv"
df_alertas.to_csv(caminho_alerta, index=False, encoding="utf-8-sig")

#Se tudo der certo, vai retornar:
print(f"onitoramento finalizado. {len(df_alertas)} alunos com alertas.")
print(f"Arquivo salvo em: {caminho_alerta}")
```

<img width="453" height="58" alt="image" src="https://github.com/user-attachments/assets/162f6d60-8bf3-41fc-bd17-88bce0c7074b" />
