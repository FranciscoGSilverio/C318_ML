# Spec T05 · Notebook, seção 4: limiar e impacto financeiro
**Status:** Concluída

## Objetivo
Responder **"Vale a pena agir?"** (pergunta 4). A seção traduz o modelo em dinheiro: escolhe o limiar de decisão que maximiza o ganho de uma campanha de retenção e compara o resultado com uma campanha aleatória e com não fazer nada. É o principal diferencial do projeto e o slide de destaque da apresentação.

## Requisitos funcionais
- RF-1: uma célula de **parâmetros** concentra as 3 premissas financeiras, que podem ser mudadas e reexecutadas (D-20):
  - valor do cliente retido = `MonthlyCharges × 12`
  - taxa de sucesso da retenção = 30%
  - custo por cliente contatado = US$ 50
- RF-2: uma função calcula o **ganho** de uma campanha:
  `Σ (valor dos churns contatados × taxa de sucesso) − (nº de contatados × custo)`.
- RF-3: o **limiar ótimo** é escolhido só com dados de treino, a partir das probabilidades *out-of-fold* (cada cliente do treino recebe a previsão de um modelo que não o viu). O teste não participa dessa escolha (D-19).
- RF-4: a **Figura 5** mostra a curva de ganho × limiar (treino, OOF), com o limiar ótimo e o limiar 0,5 marcados.
- RF-5: uma **tabela de cenários no teste** compara 5 cenários: não fazer nada; **contatar todos** (D-31); campanha aleatória com o mesmo número de contatos do modelo (valor esperado); modelo com limiar 0,5; e modelo com limiar ótimo. As colunas são: contatados, churns alcançados, ganho (US$), precisão e recall.
- RF-6: a **Figura 6** é a curva de ganho acumulado no teste: a porcentagem de churns alcançados em função da porcentagem de clientes contatados, ordenando do maior risco para o menor. Mostra a diagonal da escolha aleatória e a anotação "top 20% → X% dos churns".
- RF-7: uma **tabela de sensibilidade** (D-31) varia a taxa de sucesso {15%, 30%, 45%} e o custo por contato {US$ 25, 50, 100}. Para cada combinação, ela mostra o limiar ótimo (reescolhido no treino) e o ganho no teste do modelo, da campanha aleatória e de contatar todos.
- RF-8: uma célula de Markdown traz as **conclusões**: respostas curtas às perguntas de negócio 1 a 4, com os números do notebook.

## Entregáveis
- Seção **4 · Limiar e impacto financeiro** do `churn.ipynb`.
- `figures/05_ganho_por_limiar.png` e `figures/06_curva_ganho_acumulado.png`.
- Tabela de cenários, tabela de sensibilidade e conclusões.

## Como será feito (código resumido)

```python
# --- Parâmetros (D-20): mude aqui e reexecute ---
MESES_VALOR = 12      # valor do cliente retido = 12 mensalidades
TAXA_SUCESSO = 0.30   # 30% dos churns contatados acabam ficando
CUSTO_CONTATO = 50    # US$ por cliente contatado

def ganho(y, proba, limiar, monthly):
    contatar = proba >= limiar
    vp = contatar & (y == 1)                     # churns que a campanha alcança
    receita = (monthly[vp] * MESES_VALOR * TAXA_SUCESSO).sum()
    return receita - contatar.sum() * CUSTO_CONTATO

# MonthlyCharges saiu do modelo (D-30), mas continua no df para calcular o valor do cliente
monthly_train = df.loc[X_train.index, 'MonthlyCharges'].values
monthly_test  = df.loc[X_test.index,  'MonthlyCharges'].values

# --- Limiar ótimo com probabilidades out-of-fold do TREINO (D-19) ---
proba_oof = cross_val_predict(modelo_final, X_train, y_train, cv=cv,
                              method='predict_proba')[:, 1]
limiares = np.arange(0.05, 0.96, 0.01)
ganhos = [ganho(y_train.values, proba_oof, t, monthly_train) for t in limiares]
limiar_otimo = limiares[np.argmax(ganhos)]
# Figura 5: plot(limiares, ganhos) + linhas verticais no ótimo e em 0,5

# --- Cenários no TESTE ---
k = (proba_test >= limiar_otimo).sum()          # nº de contatos do modelo
valor_churns = (monthly_test[y_test == 1] * MESES_VALOR * TAXA_SUCESSO).sum()
aleatoria = k / len(y_test) * valor_churns - k * CUSTO_CONTATO   # valor esperado
# tabela: nada (0) | contatar todos | aleatória | modelo @0,5 | modelo @ótimo

# --- Sensibilidade (D-31): reescolhe o limiar no treino para cada premissa ---
for taxa in [0.15, 0.30, 0.45]:
    for custo in [25, 50, 100]:
        limiar = escolher_limiar(taxa, custo)          # treino (OOF)
        tabela = cenarios(limiar, taxa, custo)         # teste
        # guarda: limiar, ganho modelo, ganho aleatória, ganho contatar todos

# --- Figura 6: curva de ganho acumulado ---
ordem = np.argsort(-proba_test)                  # do maior risco para o menor
captura = np.cumsum(y_test.values[ordem]) / y_test.sum()
pct_contatados = np.arange(1, len(ordem) + 1) / len(ordem)
# plot(pct_contatados, captura) + diagonal + anotação em 20%
```

**Uma observação conceitual:** como o modelo foi treinado com `class_weight='balanced'`, as "probabilidades" saem infladas em relação à taxa real de churn. Elas servem bem para **ordenar** clientes por risco, mas não como probabilidade literal. Por isso o limiar é escolhido empiricamente pela curva de ganho, e não por uma fórmula de ponto de equilíbrio.

## Verificação
- `jupyter nbconvert --execute --inplace churn.ipynb` roda sem erro.
- O limiar ótimo vem só do treino: o teste aparece apenas na tabela de cenários e na Figura 6.
- O ganho do modelo com limiar ótimo é maior que o da campanha aleatória e maior que zero.
- Sanidade: no limiar mínimo (0,05), quase todos são contatados e o recall fica perto de 1.

## Commits previstos
1. `feat(T05): adiciona premissas financeiras e escolha do limiar ótimo (figura 5)`
2. `feat(T05): adiciona tabela de cenários no teste`
3. `feat(T05): adiciona curva de ganho acumulado (figura 6)`
4. `feat(T05): adiciona análise de sensibilidade das premissas`
5. `feat(T05): adiciona conclusões das perguntas de negócio`
6. `docs(T05): marca T05 como concluída no backlog`

Depois dos commits: **push da milestone M2** e pausa para a sua revisão (H2) da modelagem e das premissas financeiras.
