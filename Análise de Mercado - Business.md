#### Objetivo do Projeto

O projeto foi desenvolvido com o objetivo de construir uma plataforma analítica completa para academias e centros esportivos, o foco do projeto é:

- Aumentar a retenção de alunos e reduzir evasões
- Medir e acompanhar o progresso físico e engajamento
- Gerar recomendações personalizadas para treinos e nutrição
- Automatizar alertas e diagnósticos preditivos
- Entregar inteligência operacional e estratégica em dashboards e relatórios gerenciais
- Este ptojeto é uma solução ponta-a-ponta, completa, cobrindo desde a geração e ingestão de dados, até modelagem estatística, machine learning, ETL, BI e storytelling orientado ao negócio.
- Aumentar engajamento e retenção
- Melhorar a efetividade dos treinos
- Otimizar a tomada de decisão por parte da gestão

<hr>

#### Problemas Sanados

- Falta de visibilidade sobre alunos em risco de evasão
- Dificuldade de personalizar recomendações de treino
- Ausência de alertas automáticos para casos críticos
- Decisões operacionais baseadas em achismo, e não em dados

<hr>

#### Perguntas-Chave Para Serem Respondidas

- Quais alunos estão em risco de abandonar a academia?
- Como está o progresso físico dos alunos ao longo do tempo?
- Quais perfis de alunos demandam atenção específica?
- O que influencia maior perda de peso ou ganho de força?
- Como podemos automatizar a geração de recomendações personalizadas?

<hr>

#### Fonte e Qualidade dos Dados

<b>Fontes Utilizadas</b>

Simulação de dados usando as bibliotecas faker e random, foram criadas:

- Presenças em treinos (logs de máquinas/sensores)
- Feedbacks coletados via app
- Avaliações físicas (peso, força)
- Perfil demográfico dos alunos

<hr>

#### Processos de Tratamento nos Dados

<b>ETL robusto com Python, incluindo:</b>

- Limpeza e padronização
- Transformações para criação de métricas (aderência, perda de peso, ganho de força)
- Cálculo de KPIs semanais

<hr>

#### Armazenamento:

- Camada RAW no MongoDB (Data Lake)
- Banco dimensional no SQL Server com dados tratados (Data Warehouse)
- Camada analítica em CSVs com dados tratados e dados para alimentar modelos de ML

<b>Período e Granularidade:</b>

- Dados gerados para simular 12 meses de operação
- Granularidade: diária e semanal, com mais de 20 mil registros

<hr>

#### Principais KPIs e Insights Gerados

| Indicador                     | Descrição                                                        | Valor Atual (Simulado) |
| ----------------------------- | ---------------------------------------------------------------- | ---------------------- |
| **Frequência semanal**        | Quantidade de semanas em que o aluno compareceu a academia                 | 11 semanas (média)     |
| **Presença em %** | % de comparecimento em relação ao plano de treino                | 62%                    |
| **Nota média de satisfação**  | Feedback médio do aluno após os treinos                          | 3,8 / 5                |
| **Perda de peso**             | Diferença entre peso inicial e final                             | 2,6 kg (média)         |
| **Ganho de força**            | Aumento de força estimado (kg) após os treinos           | 5,1 kg (média)         |
| **Risco de evasão (%)**       | Proporção de alunos em risco de evasão baseados no modelo preditivo criado | 17%                    |
| **Segmentação de Clientes**         | Segmentação com base em comportamento e progresso dos alunos   | 5 perfis               |

<hr>

### Interpretação de Gráficos e Visualizações

#### Frequência por Semana

- Padrões: Pico no início dos meses (cheguei a conclusão que o aluno está mais motivado), e depois uma queda acentuada após 4–6 semanas (aluno mais desmotivado)
- Insight: alunos precisam de reforço e motivação no 2º e 3º mês

#### Curva de Progresso Físico (peso e força)

- Observação: progresso físico ocorre quando a frequência do aluno é contínua
- Anomalia: existem alguns alunos com nota alta e baixa evolução, sendo preciso reavaliar o plano de treino

#### Mapa de Segmentos com Clustering e PCA

- Resultado: 5 clusters distintos identificados utilizando o KMeans
- Destaque: cluster 1 significa o risco de evasão do aluno e o cluster 0 significa os alunos que estão engajados

#### Curva ROC – Classificação de Evasão

- Resultado: AUC = 0.83 mostrando ser um modelo com boa capacidade preditiva
- Importância: permite a intervenção proativa de alunos com risco de evasão

<hr>

#### Insights Estratégicos & Operacionais

<b>Desempenho Atual:</b>

- 17% dos alunos estão em risco alto de evasão
- Alunos mais frequentes têm 2 vezes mais progresso físico que os não frequentes
- Cluster 3 (novatos) precisa de mais atenção e integração
- Cluster 4 (feedbacks ruins) demanda uma revisão crucial e imediata e também de atendimento

<hr>

#### Oportunidades

- Campanhas segmentadas por perfil
- Acompanhamento nutricional personalizado
- Personal trainer oferecido para alunos de risco
- Reforço motivacional após 1º mês de treino

<hr>

#### Riscos

- Avaliações físicas desatualizadas (37% com +180 dias)
- Alunos com presença regular mas sem progresso provavelmente com o plano inadequado
- Falta de automação no follow-up dos alunos

<hr>

#### Análise Segmentada 

| Cluster | Comportamento                | Progresso | Feedback | Ação Recomendada                          |
| ------- | ---------------------------- | --------- | -------- | ----------------------------------------- |
| 0       | Frequentes e motivados       | Alto      | Médio    | Intensificar treino e metas               |
| 1       | Irregulares                  | Baixo     | Baixo    | Oferecer acompanhamento pessoal           |
| 2       | Frequentes, pouco progresso  | Baixo     | Médio    | Reavaliar treino e reforçar nutrição      |
| 3       | Novos alunos                 | Médio     | Alto     | Comunicação de boas-vindas e rotina clara |
| 4       | Frequentes com feedback ruim | Médio     | Baixo    | Conversa com instrutor e análise de acompanhamento  |

<hr>

### Recomendações Que eu Sugiro

#### Ações Imediatas

<b>Criar alertas automáticos para:</b>

- Alunos sem frequência há 14 dias
- Feedback médio menor que 3
- Avaliação física vencida
- Disparar e-mail ou WhatsApp personalizado por cluster

#### Ações de Médio Prazo

- Implantar modelo de recomendação de treino com base no cluster
- Treinar equipe para interpretação dos KPIs de performance

<hr>

#### Métricas de Acompanhamento

- % de alunos recuperados após alerta
- Aumento da frequência após recomendação
- Evolução de KPIs por cluster

<hr>

### Conclusão do Projeto

#### Resumo dos Resultados:

- Criei uma pipeline completo de BI com MongoDB + Python + SQL Server
- Modelos preditivos e segmentações foram treinados com alta precisão
- Alertas automáticos, recomendações e análises visuais foram integradas
- Mais de 20 mil registros tratados para gerar insights reais e acionáveis

<hr>

#### Impacto Iminente

- Decisões baseadas em dados, não mais em achismo
- Retenção potencializada com ações direcionadas
- Maior personalização da jornada do aluno

