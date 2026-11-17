# Spec T04 · Notebook, seção 3: modelagem
**Status:** Aguardando aprovação

## Objetivo
Responder **"Quem vai cancelar?"** (pergunta 1). A seção treina e compara 3 modelos com validação cruzada, escolhe um modelo final e mede o desempenho dele uma única vez em dados nunca vistos. O resultado é o modelo que a seção 4 usa para calcular o impacto financeiro.

> **Alteração aprovada durante a execução (D-30):** `MonthlyCharges` foi retirada das features do modelo por multicolinearidade com as colunas de serviço (R² = 0,9988), que invertia o sinal do seu coeficiente. O código abaixo já reflete a mudança.

## Requisitos funcionais
- RF-1: os dados são separados em treino (80%) e teste (20%), mantendo a proporção de churn nos dois conjuntos (estratificado).
- RF-2: um único pré-processamento é aplicado dentro do pipeline: one-hot nas categóricas e padronização nas 3 numéricas. Como ele fica dentro do pipeline, nada do teste vaza para o treino (D-15).
- RF-3: os 3 modelos (Regressão Logística como baseline, Árvore de Decisão e Random Forest) são treinados com `class_weight='balanced'`, para compensar os 26,5% de churn (D-14). Os hiperparâmetros são ajustados por `GridSearchCV` com validação cruzada estratificada de 5 folds, otimizando ROC-AUC (D-16).
- RF-4: uma **tabela comparativa** mostra, para cada modelo, o AUC médio e o desvio na validação cruzada e os melhores hiperparâmetros.
- RF-5: o **modelo final** é o de maior AUC, mas a Regressão Logística fica com a vaga se a diferença para ela for menor que 0,01, por ser mais fácil de explicar (D-16).
- RF-6: o modelo final é avaliado **no teste**, com limiar de 0,5: ROC-AUC, precisão, recall e F1 da classe churn, e a acurácia apenas como referência.
- RF-7: a **Figura 3** é a matriz de confusão no teste. A **Figura 4** mostra as 10 variáveis mais importantes do modelo final (D-17).
- RF-8: uma célula de Markdown curta interpreta a comparação, o desempenho e as variáveis mais importantes, e diz se estas confirmam a EDA.

## Entregáveis
- Seção **3 · Modelagem** do `churn.ipynb`.
- Tabela comparativa dos modelos e tabela de métricas no teste.
- `figures/03_matriz_confusao.png` e `figures/04_importancia.png`.

## Como será feito (código resumido)

```python
# --- Separação treino/teste (estratificada) ---
X, y = df.drop(columns=['Churn', 'MonthlyCharges']), df['Churn']   # D-30
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=RANDOM_STATE)

# --- Pré-processamento único (aprendido só no treino, dentro do pipeline) ---
num = ['tenure', 'TotalCharges']
cat = [c for c in X.columns if c not in num]
pre = ColumnTransformer([('cat', OneHotEncoder(handle_unknown='ignore'), cat),
                         ('num', StandardScaler(), num)])

# --- 3 modelos + grades pequenas ---
modelos = {
    'Regressão Logística': (LogisticRegression(class_weight='balanced', max_iter=1000),
                            {'clf__C': [0.01, 0.1, 1, 10]}),
    'Árvore de Decisão':   (DecisionTreeClassifier(class_weight='balanced', random_state=RANDOM_STATE),
                            {'clf__max_depth': [3, 5, 8], 'clf__min_samples_leaf': [1, 20, 50]}),
    'Random Forest':       (RandomForestClassifier(n_estimators=300, class_weight='balanced',
                                                   n_jobs=-1, random_state=RANDOM_STATE),
                            {'clf__max_depth': [None, 8, 12], 'clf__min_samples_leaf': [1, 5]}),
}
cv = StratifiedKFold(5, shuffle=True, random_state=RANDOM_STATE)
buscas = {nome: GridSearchCV(Pipeline([('pre', pre), ('clf', clf)]), grade,
                             cv=cv, scoring='roc_auc', n_jobs=-1).fit(X_train, y_train)
          for nome, (clf, grade) in modelos.items()}
# tabela: AUC médio, desvio e melhores parâmetros de cada busca

# --- Seleção (regra D-16) ---
melhor = comparacao['auc_cv'].idxmax()
if comparacao.loc[melhor, 'auc_cv'] - comparacao.loc['Regressão Logística', 'auc_cv'] < 0.01:
    melhor = 'Regressão Logística'
modelo_final = buscas[melhor].best_estimator_

# --- Avaliação única no teste (limiar 0,5) ---
proba_test = modelo_final.predict_proba(X_test)[:, 1]
pred_test = (proba_test >= 0.5).astype(int)
# roc_auc_score, precision_score, recall_score, f1_score, accuracy_score
ConfusionMatrixDisplay.from_predictions(y_test, pred_test, display_labels=['Ficou', 'Cancelou'])
# → figures/03_matriz_confusao.png

# --- Importância das variáveis do modelo final (top 10) ---
nomes = modelo_final['pre'].get_feature_names_out()
clf = modelo_final['clf']
valores = clf.coef_[0] if hasattr(clf, 'coef_') else clf.feature_importances_
# Regressão Logística: coeficiente > 0 aumenta o risco, < 0 reduz
# → barras horizontais com as 10 maiores em módulo → figures/04_importancia.png
```

## Verificação
- `jupyter nbconvert --execute --inplace churn.ipynb` roda sem erro, em poucos minutos.
- AUC no teste ≥ 0,80. O esperado para esse dataset é cerca de 0,84, com recall da classe churn em torno de 0,8 e precisão em torno de 0,5 no limiar 0,5.
- O conjunto de teste aparece só nas células de avaliação do modelo final.

## Commits previstos
1. `feat(T04): adiciona treino e comparação dos 3 modelos com validação cruzada`
2. `feat(T04): adiciona avaliação do modelo final no teste (figura 3)`
3. `feat(T04): adiciona importância das variáveis do modelo final (figura 4)`
4. `docs(T04): marca T04 como concluída no backlog`

O T04 não tem push próprio; o próximo push é na M2, depois do T05.
