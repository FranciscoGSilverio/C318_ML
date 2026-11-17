# Previsão de Churn em Operadora de Telecomunicações

**Disciplina:** C318 – Fundamentos de Machine Learning (Tópicos Especiais II)
**Instituição:** Inatel – Instituto Nacional de Telecomunicações
**Docente:** Ricardo Augusto
**Formato:** Projeto individual
**Enquadramento:** Aprendizagem Supervisionada – Classificação (binária)

> **Nota de revisão (2026-11-16):** o escopo foi enxugado durante a execução, mantendo todos os requisitos. As decisões estão em [`DECISION_LOG.md`](DECISION_LOG.md), e os resultados no [`README.md`](README.md). Destaques: `MonthlyCharges` saiu das features do modelo por multicolinearidade (D-30), e a análise financeira ganhou o cenário "contatar todos" e uma análise de sensibilidade das premissas (D-31).

---

## 1. Contexto

No setor de telecomunicações, *churn* (ou evasão) é o cancelamento do serviço pelo cliente. É um dos indicadores mais acompanhados pelas operadoras, por três motivos:

- **Conquistar um cliente novo custa bem mais do que manter um atual**, por causa de marketing, subsídio de aparelho e instalação.
- O mercado é **saturado e competitivo**: a portabilidade facilita a troca de operadora, e a diferença entre planos é pequena.
- Cada cliente perdido representa **receita recorrente perdida** durante meses ou anos.

Se a operadora consegue identificar com antecedência os clientes com maior risco de cancelar, pode agir de forma direcionada, com ofertas de retenção, upgrade de plano ou contato do suporte. Assim ela não gasta recursos com a base inteira.

## 2. Objetivos

### 2.1 Objetivo de negócio
Reduzir a perda de receita causada por cancelamentos, direcionando ações de retenção aos clientes com maior probabilidade de churn.

### 2.2 Objetivo de ciência de dados
Construir um modelo de classificação que estime a **probabilidade de churn** de cada cliente a partir de dados cadastrais, de serviços contratados e de cobrança. O modelo deve também identificar os **fatores que mais influenciam** o cancelamento.

## 3. Perguntas de Negócio

1. **Quem vai cancelar?** É possível prever, com desempenho útil, quais clientes têm maior risco de churn?
2. **Por que cancelam?** Quais características mais se associam ao cancelamento? Exemplos: tipo de contrato, tempo de casa, forma de pagamento, valor da mensalidade, serviço de internet.
3. **Quem é o cliente de risco?** Qual é o perfil típico do cliente que cancela, comparado ao que permanece?
4. **Vale a pena agir?** Supondo uma campanha de retenção com custo por cliente contatado, quanto de receita o modelo ajudaria a preservar em comparação com contatar clientes aleatoriamente?

## 4. Enquadramento do Problema de Aprendizagem

| Item | Definição |
|---|---|
| Tipo | Aprendizagem supervisionada – classificação binária |
| Variável-alvo | `Churn` (Yes / No) |
| Unidade de análise | Cliente |
| Saída do modelo | Probabilidade de churn (0 a 1) e classe prevista |
| Desafio principal | Desbalanceamento de classes (26,5% de churn) |

## 5. Dados

### 5.1 Fonte
**Telco Customer Churn**, um conjunto de dados de exemplo publicado pela IBM e amplamente usado em estudos de churn.

- **Kaggle:** https://www.kaggle.com/datasets/blastchar/telco-customer-churn
  - Arquivo: `WA_Fn-UseC_-Telco-Customer-Churn.csv`
  - O download exige uma conta gratuita no Kaggle.
- **Alternativa (GitHub da IBM):** https://github.com/IBM/telco-customer-churn-on-icp4d (pasta `data/`)

> Observação: os dados representam uma operadora fictícia da Califórnia. Isso deve ser mencionado na documentação como limitação, porque o comportamento do consumidor brasileiro pode ser diferente.

### 5.2 Visão geral
- **7.043 clientes** (linhas) e **21 colunas**
- Um único arquivo CSV, sem necessidade de junção com outras fontes

### 5.3 Dicionário de dados

| Grupo | Coluna | Descrição |
|---|---|---|
| Identificação | `customerID` | Identificador do cliente (descartar na modelagem) |
| Demográfico | `gender` | Gênero |
| | `SeniorCitizen` | Idoso (1) ou não (0) |
| | `Partner` | Possui cônjuge/parceiro |
| | `Dependents` | Possui dependentes |
| Relacionamento | `tenure` | Meses como cliente |
| | `Contract` | Tipo de contrato: mensal, 1 ano ou 2 anos |
| Serviços | `PhoneService` | Possui telefonia |
| | `MultipleLines` | Possui múltiplas linhas |
| | `InternetService` | Tipo de internet: DSL, Fibra ou nenhum |
| | `OnlineSecurity` | Serviço de segurança online |
| | `OnlineBackup` | Backup online |
| | `DeviceProtection` | Proteção de aparelho |
| | `TechSupport` | Suporte técnico |
| | `StreamingTV` | Streaming de TV |
| | `StreamingMovies` | Streaming de filmes |
| Cobrança | `PaperlessBilling` | Fatura digital |
| | `PaymentMethod` | Forma de pagamento |
| | `MonthlyCharges` | Valor mensal cobrado |
| | `TotalCharges` | Valor total cobrado até o momento |
| **Alvo** | `Churn` | Cancelou no último mês (Yes/No) |

### 5.4 Pontos de qualidade conhecidos
- `TotalCharges` vem como texto e tem **11 valores em branco**, todos de clientes com `tenure = 0` (recém-chegados). É preciso converter para número e tratar esses casos, por exemplo preenchendo com 0 ou removendo as linhas.
- `SeniorCitizen` está codificada como 0/1, enquanto as demais binárias estão como Yes/No. Convém padronizar.
- Várias colunas de serviço têm a categoria "No internet service" ou "No phone service". Vale avaliar se ela deve ser fundida com "No".
- As classes são desbalanceadas: 73,5% "No" e 26,5% "Yes".

## 6. Requisitos

### 6.1 Entregáveis (conforme escopo da disciplina)
1. **Scripts computacionais** em Python (Jupyter Notebook e/ou arquivos `.py`).
2. **Documentação do projeto**: este documento, atualizado ao longo do desenvolvimento, e o README do repositório.
3. **Apresentação final** que sintetize o projeto para públicos técnico e de negócio.

### 6.2 Requisitos funcionais do projeto
- [x] Carregar e inspecionar o conjunto de dados
- [x] Limpar e tratar inconsistências (seção 5.4)
- [x] Fazer análise exploratória com visualizações que respondam às perguntas 2 e 3
- [x] Pré-processar: codificar variáveis categóricas e escalonar as numéricas quando o modelo exigir
- [x] Separar treino e teste de forma **estratificada**
- [x] Treinar um modelo baseline e pelo menos dois modelos adicionais
- [x] Tratar o desbalanceamento de classes
- [x] Ajustar hiperparâmetros com validação cruzada
- [x] Avaliar com métricas técnicas e de negócio (seção 8)
- [x] Interpretar o modelo (importância das variáveis)
- [x] Simular o impacto financeiro de uma campanha de retenção (pergunta 4)

### 6.3 Ferramentas
| Finalidade | Ferramenta |
|---|---|
| Linguagem | Python 3.10+ |
| Ambiente | Jupyter Notebook / Google Colab / VS Code |
| Manipulação de dados | pandas, numpy |
| Visualização | matplotlib, seaborn |
| Modelagem | scikit-learn |
| Opcional | xgboost ou lightgbm, imbalanced-learn (SMOTE), shap |
| Versionamento | Git + GitHub |

## 7. Metodologia

O desenvolvimento segue as duas fases sugeridas no escopo: **análise** e **modelagem**.

### 7.1 Análise exploratória
- Taxa geral de churn
- Taxa de churn por variável categórica: contrato, internet, forma de pagamento, serviços adicionais
- Distribuição de `tenure` e `MonthlyCharges` para quem cancela e quem permanece
- Correlação entre variáveis numéricas
- Hipóteses a verificar:
  - Contratos mensais têm churn muito maior que contratos anuais
  - Clientes novos (com `tenure` baixo) cancelam mais
  - Clientes de fibra óptica cancelam mais, o que pode indicar preço ou qualidade
  - Quem não tem suporte técnico ou segurança online cancela mais
  - Pagamento por *electronic check* está associado a mais churn

### 7.2 Pré-processamento
- Remover `customerID`
- Converter `TotalCharges` para número e tratar os nulos
- Aplicar *one-hot encoding* nas categóricas
- Aplicar `StandardScaler` nas numéricas (para modelos sensíveis à escala)
- Separar treino e teste em 80/20, estratificado, com `random_state` fixo
- Montar tudo com `Pipeline` e `ColumnTransformer` do scikit-learn, para evitar vazamento de dados
- Deixar `MonthlyCharges` fora das features do modelo: ela é a soma dos serviços contratados (R² = 0,9988) e distorce os coeficientes. A coluna continua sendo usada na EDA e no cálculo financeiro (D-30)

### 7.3 Modelagem
| Modelo | Papel |
|---|---|
| Regressão Logística | Baseline interpretável |
| Árvore de Decisão | Regras fáceis de explicar ao negócio |
| Random Forest | Modelo de conjunto robusto |
| Gradient Boosting (XGBoost/LightGBM) | Opcional, costuma ter o melhor desempenho |

**Desbalanceamento:** comparar `class_weight='balanced'`, SMOTE (opcional) e o **ajuste do limiar de decisão**. O limiar costuma ser a alavanca mais útil para o negócio.

**Ajuste de hiperparâmetros:** `GridSearchCV` ou `RandomizedSearchCV` com `StratifiedKFold` de 5 folds, otimizando F1 ou AUC.

### 7.4 Interpretação
- Coeficientes da regressão logística (direção e intensidade de cada fator)
- Importância de variáveis dos modelos de árvore
- Opcional: valores SHAP para explicar previsões individuais

## 8. Métricas

### 8.1 Métricas técnicas
| Métrica | Por que importa |
|---|---|
| **Recall (classe churn)** | Mostra quantos dos clientes que vão cancelar o modelo identifica. Deixar um churn passar custa caro. |
| **Precisão (classe churn)** | Mostra quantos dos clientes sinalizados realmente cancelariam, o que evita gastar retenção à toa. |
| **F1-score** | Equilíbrio entre precisão e recall |
| **ROC-AUC** | Capacidade geral de ordenar clientes por risco, independente do limiar |
| Matriz de confusão | Base para a análise de custo |

> A **acurácia não deve ser a métrica principal**. Um modelo que prevê "não cancela" para todos já acerta 73,5%.

### 8.2 Métricas de negócio
Simulação com premissas explícitas, que podem ser ajustadas:

- **Valor de um cliente retido:** por exemplo, `MonthlyCharges × 12` meses
- **Custo da ação de retenção por cliente contatado:** por exemplo, um desconto ou bônus de valor fixo
- **Taxa de sucesso da retenção:** por exemplo, 30% dos contatados que iriam cancelar acabam ficando

Com base na matriz de confusão:

```
Ganho = (VP × taxa_sucesso × valor_cliente) − ((VP + FP) × custo_contato)
```

Indicadores a apresentar:
- Receita preservada pelo modelo **vs.** campanha aleatória **vs.** não fazer nada
- **Curva de ganho (lift):** "contatando os 20% de maior risco, alcançamos X% de todos os cancelamentos"
- Limiar de decisão que **maximiza o ganho**, que tende a ser diferente de 0,5

## 9. Estrutura do Repositório

```
C318_ML/
├── README.md                          # Resumo do projeto, resultados e como executar
├── proposta_projeto_churn_telecom.md  # Este documento
├── BACKLOG.md                         # Plano de execução por tarefas
├── DECISION_LOG.md                    # Decisões de escopo e de modelagem
├── specs/                             # Spec de cada tarefa, aprovada antes da execução
├── churn.ipynb                        # Notebook único: dados, EDA, modelagem, impacto e conclusões
├── data/
│   └── Telco-Customer-Churn.csv       # Dataset original (IBM)
├── figures/                           # Gráficos usados na apresentação
├── slides.md / slides.pdf             # Apresentação final
└── requirements.txt
```

A estrutura sugerida originalmente (4 notebooks, `src/`, `docs/`, `reports/`) foi simplificada para um notebook único e um repositório plano (D-01, D-02).

## 10. Cronograma Estimado

> **Nota de revisão:** o escopo foi reduzido ao indispensável (ver [`DECISION_LOG.md`](DECISION_LOG.md)), e o plano revisado em [`BACKLOG.md`](BACKLOG.md) estima cerca de 7–10 h de trabalho ativo do desenvolvedor, contra as 18–30 h abaixo.

O projeto será desenvolvido por **1 pessoa com apoio de desenvolvimento agêntico (Claude Code)**. O agente escreve e executa a maior parte do código. O desenvolvedor fica com a revisão, a interpretação dos resultados, as decisões de modelagem e de negócio, a documentação e a apresentação.

As estimativas abaixo são de **horas de trabalho ativo do desenvolvedor**.

| Etapa | Papel do agente | Papel do desenvolvedor | Horas ativas |
|---|---|---|---|
| 1. Setup | Criar o repositório, a estrutura de pastas, o `requirements.txt` e o README inicial | Baixar o dataset (Kaggle) e escrever o `CLAUDE.md` com o contexto do projeto | 1–2 h |
| 2. Limpeza + EDA | Gerar o notebook de tratamento, os gráficos e os testes de hipóteses | Interpretar os gráficos, validar as hipóteses e registrar os achados | 3–5 h |
| 3. Modelagem | Montar os pipelines, os modelos, a validação cruzada, o ajuste de hiperparâmetros e o tratamento do desbalanceamento | Entender as escolhas técnicas, comparar os modelos e definir o modelo final | 4–6 h |
| 4. Interpretação + impacto financeiro | Calcular a importância das variáveis, SHAP, curva de ganho e simulação de limiar | Definir e justificar as premissas financeiras e tirar as conclusões de negócio | 2–4 h |
| 5. Documentação | Atualizar o README e consolidar os resultados | Revisar o conteúdo e o tom | 1–2 h |
| 6. Apresentação | Gerar o rascunho dos slides e exportar as figuras | Refinar a narrativa e ensaiar | 4–6 h |
| 7. Domínio do conteúdo | Explicar o código e os conceitos sob demanda | Estudar para responder perguntas sem depender do agente | 3–5 h |
| **Total** | | | **~18–30 h** |

Para referência, sem assistência agêntica o mesmo escopo costuma levar de 40 a 60 horas.

### 10.1 Prazo em calendário

| Ritmo de dedicação | Prazo estimado |
|---|---|
| Intensivo (~4 h/dia) | 1 semana a 10 dias |
| Moderado (~8 h/semana) | 3–4 semanas |
| Leve (~4 h/semana) | 5–7 semanas |

### 10.2 Observações
- **Partes que não comprimem com o agente:** as sessões de mentoria (dependem da agenda do professor), o ensaio da apresentação e a compreensão do conteúdo.
- **Mentorias planejadas:** uma após a EDA (etapa 2) e outra após a modelagem (etapa 3).
- **Folga:** reservar cerca de 20% a mais no prazo para imprevistos e ajustes pedidos nas mentorias.
- **Boas práticas do fluxo agêntico:** trabalhar uma etapa por vez, fazer commit no git ao fim de cada etapa, pedir explicações das escolhas técnicas e conferir números de controle (taxa de churn de cerca de 26,5% e 11 nulos em `TotalCharges`).
- **Transparência:** o uso de IA no desenvolvimento será declarado na documentação, conforme a orientação do professor.

## 11. Riscos e Limitações

| Risco / limitação | Mitigação |
|---|---|
| Dataset fictício e de outro país | Declarar a limitação e discutir como o modelo seria adaptado a uma operadora real |
| Dataset muito conhecido | Diferenciar-se com a análise de impacto financeiro e a escolha do limiar |
| Dados de um único momento (sem histórico) | Ressaltar que, em produção, o modelo seria retreinado periodicamente |
| Variáveis com pouco poder preditivo | Focar a narrativa nos fatores mais relevantes encontrados na EDA |
| Valores monetários em USD | Manter em USD ou converter com uma taxa fixa declarada |

## 12. Roteiro da Apresentação Final

1. **O problema:** o custo do churn para uma operadora (1 slide)
2. **Os dados:** quem são os clientes (1 slide)
3. **Principais achados da EDA:** de 3 a 4 gráficos que respondem "por que cancelam?"
4. **O modelo:** abordagem, comparação entre modelos e métrica escolhida
5. **Desempenho:** matriz de confusão e curva de ganho, em linguagem simples
6. **Impacto financeiro:** quanto a operadora economizaria (slide de destaque)
7. **Recomendações de negócio:** ações concretas baseadas nos fatores de risco
8. **Limitações e próximos passos**

> Adapte a linguagem ao público. Na parte técnica, use métricas e validação. Na parte de negócio, use clientes, reais e ações.

## 13. Referências

- IBM. *Telco Customer Churn* (dataset de exemplo). Disponível em: https://github.com/IBM/telco-customer-churn-on-icp4d
- Kaggle. *Telco Customer Churn*. Disponível em: https://www.kaggle.com/datasets/blastchar/telco-customer-churn
- Documentação do scikit-learn: https://scikit-learn.org/stable/
- Géron, A. *Mãos à Obra: Aprendizado de Máquina com Scikit-Learn, Keras & TensorFlow*. Alta Books.
