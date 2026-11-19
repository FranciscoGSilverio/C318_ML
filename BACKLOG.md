# Backlog de Execução

Plano para que um agente de codificação execute o projeto **Previsão de Churn em Telecom** com o menor esforço que ainda cumpra os requisitos da [proposta](proposta_projeto_churn_telecom.md).
As justificativas de cada escolha estão no [`DECISION_LOG.md`](DECISION_LOG.md) (referências `D-xx`).

## Instruções para o agente

1. Execute as tarefas **em ordem**, uma por vez. Cada tarefa tem entregável e critério de aceite.
2. **Spec antes de cada tarefa (D-28):** antes de escrever qualquer código de uma tarefa, crie `specs/T0x_<nome>.md` a partir do [modelo de spec](#modelo-de-spec), faça o commit e **pare até o aluno aprovar**. Ajustes pedidos pelo aluno entram na spec antes da execução.
3. **Não implemente nada da seção "Fora de escopo"**, nem itens novos não listados aqui. Se algo parecer necessário, registre uma nova decisão no `DECISION_LOG.md` e pare para pedir aprovação ao aluno.
4. **Commits granulares (D-29):** um commit por mudança lógica (por exemplo, uma seção do notebook ou um arquivo de configuração), nunca um commit único para a tarefa inteira. Mensagens no formato `<tipo>(T0x): <resumo>`, com tipo `spec`, `feat`, `docs`, `chore` ou `data`. A mensagem tem **uma única linha descritiva**, sem corpo, sem `Co-Authored-By` e sem mencionar ferramentas de IA. Ao concluir a tarefa, marque o checkbox aqui e altere o status da spec para "Concluída".
5. **Push a cada milestone (D-29):** faça `git push` ao concluir cada milestone da tabela abaixo.
6. Nos pontos marcados com 🧑 **Revisão do aluno**, pare e aguarde.
7. Ambiente: Python 3.10, venv em `.venv/`. Todos os comandos rodam com o venv ativo.

### Milestones (push)

| Milestone | Concluída quando |
|---|---|
| M0 · Planejamento | Backlog, decision log e spec do T01 commitados |
| M1 · Dados e EDA | T01–T03 concluídas |
| M2 · Modelo e impacto | T04–T05 concluídas |
| M3 · Documentação | T06 concluída |
| M4 · Apresentação | T07 concluída (entrega final) |

### Modelo de spec

Mínima, focada no resultado e no entregável. Deve permitir que o aluno entenda **o que** será feito e **como**.

```markdown
# Spec T0x · <nome>
**Status:** Aguardando aprovação | Aprovada | Concluída

## Objetivo
1–2 frases: qual resultado a tarefa produz e por que o projeto precisa dele.

## Requisitos funcionais
- RF-1 … (o que deve existir ou funcionar ao final; sem detalhes de implementação)

## Entregáveis
Arquivos, seções, figuras e tabelas produzidos.

## Como será feito (código resumido)
Esqueleto do código, com as linhas essenciais e comentários explicando cada passo.
Não é o código final completo.

## Verificação
Como confirmar que funcionou (comandos e valores esperados).

## Commits previstos
Lista dos commits granulares.
```

## Estrutura final do repositório

```
.
├── BACKLOG.md
├── DECISION_LOG.md
├── README.md
├── proposta_projeto_churn_telecom.md
├── requirements.txt
├── .gitignore
├── churn.ipynb            # único notebook, versionado com saídas (D-01, D-26)
├── specs/                 # uma spec por tarefa, aprovada antes da execução (D-28)
├── data/
│   └── Telco-Customer-Churn.csv
└── figures/               # PNGs gerados pelo notebook e usados nos slides
```

---

## Tarefas do agente

### T01 · Setup do ambiente e dos dados
- [x] Criar `.gitignore` com `.venv/`, `.ipynb_checkpoints/` e `__pycache__/`.
- [x] Criar o venv e instalar `pandas scikit-learn matplotlib jupyter` (D-04).
- [x] Gerar `requirements.txt` com **apenas essas 4 bibliotecas**, fixadas nas versões instaladas (`==`). Não usar `pip freeze` completo.
- [x] Baixar o dataset (D-03):
  `curl -L -o data/Telco-Customer-Churn.csv https://raw.githubusercontent.com/IBM/telco-customer-churn-on-icp4d/master/data/Telco-Customer-Churn.csv`

**Entregável:** `.gitignore`, `requirements.txt` e `data/Telco-Customer-Churn.csv`.
**Aceite:** `python -c "import pandas, sklearn, matplotlib"` roda sem erro, e o CSV tem 7.043 linhas × 21 colunas.

### T02 · Notebook, seção 1: dados e limpeza
Criar `churn.ipynb` com:
- [x] Célula de imports e `RANDOM_STATE = 42`, além de `os.makedirs('figures', exist_ok=True)`.
- [x] Carga do CSV e remoção de `customerID`.
- [x] `TotalCharges` convertida para número (`pd.to_numeric(errors='coerce')`), com os brancos preenchidos com 0 (D-08).
- [x] `Churn` convertido para 0/1.
- [x] Checagens de controle (D-07): `assert` de 7.043 linhas, 11 nulos em `TotalCharges` antes do preenchimento e taxa de churn entre 0,26 e 0,27.

**Entregável:** seção 1 do notebook.
**Aceite:** a seção executa sem erro e todos os `assert` passam.
**Não fazer:** padronizar `SeniorCitizen` nem fundir "No internet service" com "No" (D-09).

### T03 · Notebook, seção 2: análise exploratória
- [x] Taxa geral de churn (1 número).
- [x] **Figura 1** (`figures/01_churn_por_categoria.png`): grade de gráficos de barras com a taxa de churn por categoria, para `Contract`, `InternetService`, `PaymentMethod`, `TechSupport`, `OnlineSecurity` e `PaperlessBilling`, gerada num único laço.
- [x] **Figura 2** (`figures/02_numericas.png`): boxplots de `tenure` e `MonthlyCharges` por classe de churn.
- [x] Tabela de perfil (pergunta 3): `groupby('Churn')` com a média das numéricas e a moda de `Contract`, `InternetService` e `PaymentMethod`.
- [x] Célula Markdown com o **veredito das 5 hipóteses** da seção 7.1 (confirmada ou não), cada uma com o número que a sustenta (D-10). Inclua uma frase sobre `TotalCharges` ≈ `tenure × MonthlyCharges` (D-11).

**Entregável:** seção 2 do notebook, 2 figuras e a tabela de hipóteses.
**Aceite:** as figuras existem em `figures/` e o texto das hipóteses usa números calculados no próprio notebook.

> 🧑 **Revisão do aluno (H1):** ler a seção 2 e validar as interpretações antes de seguir.

### T04 · Notebook, seção 3: modelagem
- [x] Divisão treino/teste 80/20 com `stratify=y` e `random_state=RANDOM_STATE`.
- [x] Um `ColumnTransformer` compartilhado: `OneHotEncoder(handle_unknown='ignore')` nas categóricas e `StandardScaler` nas numéricas (D-15).
- [x] Três pipelines, todos com `class_weight='balanced'` (D-12, D-14), e `GridSearchCV` com `StratifiedKFold(5, shuffle=True, random_state=RANDOM_STATE)` e `scoring='roc_auc'` (D-16). Grades sugeridas:
  - Regressão Logística: `C ∈ {0.01, 0.1, 1, 10}`, `max_iter=1000`
  - Árvore de Decisão: `max_depth ∈ {3, 5, 8}` × `min_samples_leaf ∈ {1, 20, 50}`
  - Random Forest: `n_estimators=300`, `max_depth ∈ {None, 8, 12}` × `min_samples_leaf ∈ {1, 5}`, `n_jobs=-1`
- [x] Tabela comparativa: AUC médio ± desvio da validação cruzada para cada modelo.
- [x] Seleção do modelo final pela regra de D-16: maior AUC médio, mas se a diferença para a Regressão Logística for menor que 0,01, fica a Regressão Logística.
- [x] Avaliação do modelo final **no teste**, com limiar 0,5: ROC-AUC, precisão, recall e F1 da classe churn, e acurácia apenas como referência.
- [x] **Figura 3** (`figures/03_matriz_confusao.png`): matriz de confusão no teste.
- [x] **Figura 4** (`figures/04_importancia.png`): as 10 variáveis mais importantes do modelo final, com coeficientes se for a Regressão Logística ou `feature_importances_` se for árvore (D-17). Use os nomes vindos de `get_feature_names_out()`.

**Entregável:** seção 3 do notebook, tabela comparativa e figuras 3 e 4.
**Aceite:** AUC no teste ≥ 0,80 (é o valor esperado para esse dataset; se ficar abaixo, investigar antes de seguir). O conjunto de teste é usado só para o modelo final.

### T05 · Notebook, seção 4: limiar e impacto financeiro
- [x] Célula de **parâmetros** (D-20): `MESES_VALOR = 12`, `TAXA_SUCESSO = 0.30` e `CUSTO_CONTATO = 50`.
- [x] Função `ganho(y, proba, limiar, monthly)` que calcula `Σ(MonthlyCharges × MESES_VALOR × TAXA_SUCESSO dos VP) − (VP + FP) × CUSTO_CONTATO`.
- [x] Probabilidades out-of-fold no treino com `cross_val_predict(modelo_final, X_train, y_train, cv=5, method='predict_proba')` e varredura de limiares de 0,05 a 0,95 em passos de 0,01 para achar o limiar ótimo (D-19).
- [x] **Figura 5** (`figures/05_ganho_por_limiar.png`): curva de ganho × limiar no treino (OOF), com o limiar ótimo marcado.
- [x] Tabela de cenários **no teste** (D-22): não fazer nada (US$ 0), contatar todos (D-31), campanha aleatória com o mesmo número de contatos (valor esperado), modelo com limiar 0,5 e modelo com o limiar ótimo. Para cada cenário: clientes contatados, churns alcançados, ganho em US$, precisão e recall.
- [x] **Figura 6** (`figures/06_curva_ganho_acumulado.png`): curva de ganho acumulado no teste (% de churns alcançados × % de clientes contatados, ordenados por risco), com a diagonal aleatória e uma anotação "top 20% → X% dos churns".
- [x] Tabela de sensibilidade (D-31): taxa de sucesso {15%, 30%, 45%} × custo {US$ 25, 50, 100}, com o limiar reescolhido no treino e o ganho no teste do modelo, da campanha aleatória e de contatar todos.
- [x] Célula Markdown com as conclusões: respostas curtas para as perguntas de negócio 1 a 4, usando os números do notebook.

**Entregável:** seção 4 do notebook, figuras 5 e 6, tabela de cenários e conclusões.
**Aceite:** o limiar ótimo foi escolhido sem usar o conjunto de teste, e o ganho do modelo com limiar ótimo é maior que o da campanha aleatória.

### T06 · Execução limpa, README e atualização da proposta
- [x] Executar o notebook de ponta a ponta: `jupyter nbconvert --to notebook --execute --inplace churn.ipynb` (D-26).
- [x] Criar `README.md` (no máximo uma página) com: objetivo; dataset e fonte; como executar (`python -m venv .venv`, `pip install -r requirements.txt`, abrir `churn.ipynb`); resultados principais (modelo escolhido, AUC, recall, limiar, ganho contra a campanha aleatória, top 20% → X%); 3 a 5 principais fatores de churn; limitações (seção 11 da proposta); seção "Uso de IA" (D-27); links para `DECISION_LOG.md` e `slides.pdf`.
- [x] Atualizar a proposta (D-25): taxa de churn igual a **26,5%** em todo o documento (seções 4 e 5.4), checklist 6.2 marcado, seção 9 com a estrutura final deste backlog e seção 10 com uma nota de que o escopo foi reduzido conforme o `DECISION_LOG.md`.

**Entregável:** notebook executado e versionado com saídas, `README.md` e proposta atualizada.
**Aceite:** o `nbconvert` termina sem erro e todos os números do README batem com as saídas do notebook.

### T07 · Apresentação
> **Concluída:** a apresentação foi entregue em vídeo (link no `README.md`). No formato final, 6 slides servem de apoio para uma demonstração do notebook no VS Code. Os slides, o PDF e o roteiro ficam fora do repositório, que guarda só o código e a documentação.

- [x] Criar `slides.md` no formato Marp (`marp: true` no front matter) com **8 slides** seguindo o roteiro da seção 12 da proposta, usando as figuras de `figures/` (D-23):
  1. O problema · 2. Os dados · 3. Achados da EDA (figuras 1 e 2) · 4. O modelo (tabela comparativa) · 5. Desempenho (figuras 3 e 6) · 6. Impacto financeiro (tabela de cenários, slide de destaque) · 7. Recomendações de negócio (figura 4) · 8. Limitações e próximos passos
- [x] Incluir a nota de modelagem da **D-30** (multicolinearidade de `MonthlyCharges`) no slide 4 ou 7, com a mensagem de negócio e a lição "prever bem não é o mesmo que explicar bem". O texto pronto está no `DECISION_LOG.md`.
- [x] Notas do apresentador em cada slide, com 2 a 4 tópicos no formato `<!-- ... -->` (D-24), incluindo as perguntas prováveis da banca e a resposta curta de cada uma.
- [x] Exportar com `npx @marp-team/marp-cli slides.md --pdf --allow-local-files -o slides.pdf`. Se falhar (por exemplo, por falta de Chromium no WSL), não insistir: deixar a exportação para o aluno (H3) e registrar isso no commit.

**Entregável:** `slides.md` e, se a exportação funcionar, `slides.pdf`.
**Aceite:** 8 slides, todas as figuras referenciadas existem e os números batem com o notebook.

---

## Tarefas do aluno

| ID | Quando | Tarefa | Tempo estimado |
|---|---|---|---|
| H1 | Após T03 | Revisar a EDA e validar as interpretações das hipóteses | 0,5–1 h |
| H2 | Após T05 | Revisar a escolha do modelo, as premissas financeiras (ajustar a célula de parâmetros se quiser) e as conclusões | 1–1,5 h |
| H3 | Após T07 | Revisar README e slides; exportar o PDF pela extensão Marp do VS Code, se o T07 não exportou; ensaiar com as notas do apresentador | 2–3 h |
| H0 | Antes de cada tarefa | Revisar e aprovar a spec da tarefa | 10–15 min por spec |
| H4 | Final | Estudar o notebook para responder perguntas sem o agente; entregar o link do repositório | 2–3 h |
| — | Se exigido pela disciplina | Mentorias com o professor: após H1 (EDA) e após H2 (modelagem) | agenda do professor |

**Total estimado para o aluno: cerca de 7–10 h ativas, incluindo a revisão das specs**, contra 18–30 h no cronograma original da proposta. O tempo de execução do agente (T01–T07) fica na ordem de 1–2 h, sem contar as pausas para revisão.

---

## Fora de escopo (não implementar)

| Item | Decisão |
|---|---|
| Quatro notebooks separados; pastas `src/`, `docs/`, `reports/` | D-01, D-02 |
| Download pelo Kaggle | D-03 |
| seaborn, xgboost, lightgbm, imbalanced-learn, shap | D-04, D-05, D-13, D-14, D-17 |
| `CLAUDE.md` | D-06 |
| Testes unitários, pytest, CI | D-07 |
| Padronizar `SeniorCitizen`; fundir "No internet/phone service" | D-09 |
| Testes estatísticos formais; matriz de correlação | D-10, D-11 |
| SMOTE; `RandomizedSearchCV`; grades amplas | D-14, D-16 |
| SHAP; interpretação de todos os modelos | D-17 |
| Salvar modelo, API, deploy, dados intermediários | D-01, D-18 |
| Conversão para reais | D-21 |
| python-pptx; documento separado de perguntas e respostas | D-23, D-24 |
