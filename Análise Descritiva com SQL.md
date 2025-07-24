#### Importando Bibliotecas e Conectando ao DW

```python
import pyodbc
import pandas as pd
import warnings
warnings.filterwarnings("ignore")

conn = pyodbc.connect(
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=(localdb)\MSSQLLocalDB;'
    r'DATABASE=AcademiaDW;'
    r'Trusted_Connection=yes;'
)
```

#### Formatação nas Colunas para as Querys

```python
#Formatação paras as colunas nas querys.
def executar_query(sql, formatacoes=None):
    df = pd.read_sql(sql, conn)

    if formatacoes:
        for col, tipo in formatacoes.items():
            if col in df.columns:
                if tipo == "int":
                    df[col] = df[col].fillna(0).astype(int).map('{:,}'.format).str.replace(',', '.')
                elif tipo == "float":
                    df[col] = df[col].fillna(0).round(2).map(lambda x: f"{x:,.2f}".replace(",", "X").replace(".", ",").replace("X", "."))
                elif tipo == "money":
                    df[col] = df[col].fillna(0).round(2).map(lambda x: f"R$ {x:,.2f}".replace(",", "X").replace(".", ",").replace("X", "."))
                elif tipo == "decimal":
                    df[col] = df[col].fillna(0).round(4).map(lambda x: f"{x:,.4f}".replace(",", "X").replace(".", ",").replace("X", "."))
                elif tipo == "percent":
                    df[col] = df[col].fillna(0).round(2).map(lambda x: f"{x:.2%}".replace(".", ","))
    return df
```

#### Status na Academia por Alunos

```python
sql_1 = """
SELECT status, COUNT(*) AS total_alunos
FROM dim_usuario
GROUP BY status;
"""
df_1 = executar_query(sql_1, {"total_alunos": "int"})
print(df_1)
```

<img width="220" height="86" alt="image" src="https://github.com/user-attachments/assets/acdfa9c3-d7cb-4e03-aa94-6e0979ea78a7" />

#### Frequência Média Semanal

```pythonn
sql_2 = """
SELECT AVG(total_presencas) AS freq_media_semanal
FROM fato_frequencia;
"""
df_2 = executar_query(sql_2, {"freq_media_semanal": "float"})
print(df_2)
```

<img width="180" height="47" alt="image" src="https://github.com/user-attachments/assets/eaa518e9-2d82-45ec-b24f-058306bb0656" />

<b>Observação:</b> Esse resultado é confuso, porém na fonte de dados só existe dois dados para a frequência semanal, 1 ou 2, o que o resultado mostra é que geralmente existe a frequência semanal média de apenas 1 (uma) vez na semana.

#### Semanas Prestes por Quantidade de Alunos

```python
sql_3 = """
SELECT semanas_presentes, COUNT(id_usuario) AS qtd_alunos
FROM fato_final_aluno
GROUP BY semanas_presentes
ORDER BY semanas_presentes DESC;
"""
df_3 = executar_query(sql_3, {"semanas_presentes": "int", "qtd_alunos": "int"})
print(df_3)
```

<img width="299" height="486" alt="image" src="https://github.com/user-attachments/assets/833183ab-1475-404e-89ea-68a6fabdc840" />

#### Média de Nota por Alunos

```python
sql_4 = """
SELECT id_usuario, media_nota
FROM fato_final_aluno
ORDER BY media_nota DESC;
"""
df_4 = executar_query(sql_4, {"id_usuario": "int", "media_nota": "float"})
print(df_4.head(10))
```

<img width="213" height="216" alt="image" src="https://github.com/user-attachments/assets/2d99c0a5-aca0-46c7-b2db-dedc120febed" />

#### Top 10 Alunos por Perda de Peso 

```python
sql_5 = """
SELECT id_usuario, perda_peso
FROM fato_final_aluno
ORDER BY perda_peso DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;
"""
df_5 = executar_query(sql_5, {"id_usuario": "int", "perda_peso": "float"})
print(df_5)
```

<img width="204" height="219" alt="image" src="https://github.com/user-attachments/assets/1b949cf8-12fb-495d-b933-b7011d205e88" />

#### Ganho Médio de Força Por Faixa Etária

```python
sql_6 = """
SELECT
  CASE 
    WHEN idade BETWEEN 18 AND 25 THEN '18-25'
    WHEN idade BETWEEN 26 AND 35 THEN '26-35'
    WHEN idade BETWEEN 36 AND 45 THEN '36-45'
    WHEN idade BETWEEN 46 AND 55 THEN '46-55'
    ELSE '56+' 
  END AS faixa_idade,
  AVG(ganho_forca) AS ganho_medio_forca
FROM fato_final_aluno
GROUP BY 
  CASE 
    WHEN idade BETWEEN 18 AND 25 THEN '18-25'
    WHEN idade BETWEEN 26 AND 35 THEN '26-35'
    WHEN idade BETWEEN 36 AND 45 THEN '36-45'
    WHEN idade BETWEEN 46 AND 55 THEN '46-55'
    ELSE '56+' 
  END
ORDER BY faixa_idade;
"""
df_6 = executar_query(sql_6, {"ganho_medio_forca": "float"})
print(df_6)
```

<img width="294" height="133" alt="image" src="https://github.com/user-attachments/assets/19114c1e-483f-462d-991c-5a452914816f" />

#### Taxa de Evasão Por Mês e Ano

```python
sql_7 = """
SELECT FORMAT(data_inicio, 'yyyy-MM') AS mes_ano,
       COUNT(CASE WHEN status IN ('Inativo', 'Cancelado') THEN 1 END) AS alunos_evasao,
       COUNT(*) AS total_alunos,
       CAST(COUNT(CASE WHEN status IN ('Inativo', 'Cancelado') THEN 1 END) * 1.0 / COUNT(*) AS DECIMAL(5,4)) AS taxa_evasao
FROM dim_usuario
GROUP BY FORMAT(data_inicio, 'yyyy-MM')
ORDER BY mes_ano;
"""
df_7 = executar_query(sql_7, {"alunos_evasao": "int", "total_alunos": "int", "taxa_evasao": "decimal"})
print(df_7)
```

<img width="418" height="511" alt="image" src="https://github.com/user-attachments/assets/c589e80a-21b7-4143-ace0-fb51960fd148" />

#### Top 10 Alunos com Mais Feedbacks

```python
sql_8 = """
SELECT id_usuario, total_feedbacks
FROM fato_final_aluno
ORDER BY total_feedbacks DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;
"""
df_8 = executar_query(sql_8, {"id_usuario": "int", "total_feedbacks": "int"})
print(df_8)
```

<img width="244" height="218" alt="image" src="https://github.com/user-attachments/assets/fb594e57-03b6-4da7-b987-dbc92ac7434f" />

#### Correlação de Presença com Nota Média de Feedback

```python
sql_9 = """
SELECT semanas_presentes, AVG(media_nota) AS media_feedback
FROM fato_final_aluno
GROUP BY semanas_presentes
ORDER BY semanas_presentes DESC;
"""
df_9 = executar_query(sql_9, {"semanas_presentes": "int", "media_feedback": "float"})
print(df_9)
```

<img width="298" height="493" alt="image" src="https://github.com/user-attachments/assets/bf367e1a-a9fc-442d-88de-690ec36c9211" />

#### Perda ou Ganho Médio de Peso por Gênero

```python
sql_10 = """
SELECT genero, AVG(perda_peso) AS media_perda_peso
FROM fato_final_aluno
GROUP BY genero;
"""
df_10 = executar_query(sql_10, {"media_perda_peso": "float"})
print(df_10)
```

<img width="243" height="69" alt="image" src="https://github.com/user-attachments/assets/2798d005-33f7-43ef-a105-4b362959b306" />

#### Ganho ou Perda Média de Força por Status na Academia

```python
sql_11 = """
SELECT status, AVG(ganho_forca) AS media_ganho_forca
FROM fato_final_aluno
GROUP BY status;
"""
df_11 = executar_query(sql_11, {"media_ganho_forca": "float"})
print(df_11)
```

<img width="267" height="92" alt="image" src="https://github.com/user-attachments/assets/2ae6774c-22d5-4f95-8912-305c3e15dd06" />

#### Percentual de Aderência - Presenças

```python
sql_12 = """
SELECT 
    CAST(SUM(CASE WHEN semanas_presentes > 10 THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS percentual_aderencia_alta
FROM fato_final_aluno;
"""
df_12 = executar_query(sql_12, {"percentual_aderencia_alta": "float"})
print(df_12)
```

<img width="233" height="46" alt="image" src="https://github.com/user-attachments/assets/f86e189d-c50b-4633-8505-5a8683aafbc0" />

