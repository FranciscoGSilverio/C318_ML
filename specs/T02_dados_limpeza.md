# Spec T02 · Notebook, seção 1: dados e limpeza
**Status:** Concluída

## Objetivo
Criar o `churn.ipynb` com a primeira seção, que carrega o CSV e entrega um DataFrame `df` limpo e conferido. Todas as seções seguintes (EDA, modelagem e impacto) partem desse `df`.

## Requisitos funcionais
- RF-1: o notebook carrega `data/Telco-Customer-Churn.csv` e remove `customerID`, que é só um identificador e não tem valor preditivo.
- RF-2: `TotalCharges` vira coluna numérica. Os 11 valores em branco são preenchidos com 0, porque são clientes com `tenure = 0` que ainda não pagaram nada (D-08).
- RF-3: o alvo `Churn` vira 0/1 (1 = cancelou), o formato que os modelos e o cálculo de taxas usam.
- RF-4: as checagens de controle (D-07) param o notebook com erro se os dados não forem os esperados: 7.043 linhas, 11 brancos, todos com `tenure = 0`, e taxa de churn entre 26% e 27%.
- RF-5: existe uma constante `RANDOM_STATE = 42`, usada em todas as etapas aleatórias seguintes, e a pasta `figures/` é criada.

## Entregáveis
- `churn.ipynb` com título e **Seção 1 · Dados e limpeza** (4 células de código e 1 de Markdown).
- Saída visível: as 5 primeiras linhas do `df` e a taxa de churn impressa (esperado: 26,5%).

## Como será feito (código resumido)

```python
# --- Célula 1: configuração ---
import os
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

RANDOM_STATE = 42                      # mesma semente em todo o projeto → resultados reproduzíveis
os.makedirs('figures', exist_ok=True)  # destino das figuras usadas nos slides

# --- Célula 2: carga ---
df = pd.read_csv('data/Telco-Customer-Churn.csv')
df = df.drop(columns='customerID')     # identificador, não é variável preditiva
df.head()

# --- Célula 3: limpeza ---
# TotalCharges vem como texto; os brancos (" ") viram NaN na conversão
df['TotalCharges'] = pd.to_numeric(df['TotalCharges'], errors='coerce')
nulos = df['TotalCharges'].isna()
n_nulos = nulos.sum()
tenure_dos_nulos = df.loc[nulos, 'tenure']
df['TotalCharges'] = df['TotalCharges'].fillna(0)   # D-08: cliente novo, ainda não pagou

df['Churn'] = (df['Churn'] == 'Yes').astype(int)    # alvo binário: 1 = cancelou

# --- Célula 4: checagens de controle (D-07) ---
assert len(df) == 7043, 'número de clientes inesperado'
assert n_nulos == 11, 'número de brancos em TotalCharges inesperado'
assert (tenure_dos_nulos == 0).all(), 'brancos deveriam ser só clientes novos'
assert 0.26 < df['Churn'].mean() < 0.27, 'taxa de churn fora do esperado'
print(f"Clientes: {len(df)} | Taxa de churn: {df['Churn'].mean():.1%}")
```

A célula de Markdown no topo da seção explica em 2–3 linhas as decisões de limpeza e o que **não** foi feito (D-09: `SeniorCitizen` e "No internet service" ficam como estão).

## Verificação
```bash
jupyter nbconvert --to notebook --execute --inplace churn.ipynb   # roda sem erro (todos os asserts passam)
```
Saída esperada da célula 4: `Clientes: 7043 | Taxa de churn: 26.5%`.

## Commits previstos
1. `feat(T02): cria churn.ipynb com seção 1 (dados e limpeza)`
2. `docs(T02): marca T02 como concluída no backlog`
