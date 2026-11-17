# Spec T03 · Notebook, seção 2: análise exploratória
**Status:** Concluída

## Objetivo
Responder com dados a duas perguntas de negócio: **"Por que cancelam?"** (pergunta 2) e **"Quem é o cliente de risco?"** (pergunta 3). Para isso, cada uma das 5 hipóteses da proposta recebe um veredito baseado em números. Os gráficos produzidos aqui vão direto para o slide 3 da apresentação.

## Requisitos funcionais
- RF-1: o notebook mostra a taxa geral de churn, que é a linha de referência de todos os gráficos.
- RF-2: a **Figura 1** mostra a taxa de churn por categoria para as 6 variáveis ligadas às hipóteses: `Contract`, `InternetService`, `PaymentMethod`, `TechSupport`, `OnlineSecurity` e `PaperlessBilling`.
- RF-3: a **Figura 2** compara `tenure` e `MonthlyCharges` entre quem cancela e quem fica.
- RF-4: uma **tabela de perfil** mostra o cliente típico de cada classe: médias das numéricas e categoria mais comum de contrato, internet e pagamento.
- RF-5: uma **tabela de hipóteses**, calculada em código, mostra para cada hipótese a taxa de churn do grupo de risco, a do grupo de comparação, a razão entre elas e o veredito. A hipótese é **confirmada** quando o grupo de risco tem pelo menos 1,5× o churn do grupo de comparação.
- RF-6: uma célula de Markdown interpreta os resultados em linguagem de negócio, com respostas curtas às perguntas 2 e 3 e uma frase sobre `TotalCharges` ≈ `tenure × MonthlyCharges` (D-11).

## Entregáveis
- Seção **2 · Análise exploratória** do `churn.ipynb`.
- `figures/01_churn_por_categoria.png` e `figures/02_numericas.png`.
- Tabela de perfil e tabela de hipóteses, visíveis no notebook.

## Como será feito (código resumido)

```python
taxa_geral = df['Churn'].mean()

# --- Figura 1: taxa de churn por categoria (um laço, 6 gráficos) ---
cats = ['Contract', 'InternetService', 'PaymentMethod',
        'TechSupport', 'OnlineSecurity', 'PaperlessBilling']
fig, axes = plt.subplots(2, 3, figsize=(15, 8))
for ax, col in zip(axes.flat, cats):
    df.groupby(col)['Churn'].mean().sort_values().plot.barh(ax=ax)
    ax.axvline(taxa_geral, ls='--', color='gray')   # referência: média geral
    ax.set_title(col)
fig.savefig('figures/01_churn_por_categoria.png', dpi=150, bbox_inches='tight')

# --- Figura 2: distribuições numéricas por classe ---
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
for ax, col in zip(axes, ['tenure', 'MonthlyCharges']):
    df.boxplot(column=col, by='Churn', ax=ax)
fig.savefig('figures/02_numericas.png', dpi=150, bbox_inches='tight')

# --- Perfil: cliente típico de cada classe ---
moda = lambda s: s.mode()[0]
perfil = df.groupby('Churn').agg(
    tenure=('tenure', 'mean'), MonthlyCharges=('MonthlyCharges', 'mean'),
    Contract=('Contract', moda), InternetService=('InternetService', moda),
    PaymentMethod=('PaymentMethod', moda))

# --- Hipóteses: grupo de risco vs. grupo de comparação ---
def taxa(mask): return df.loc[mask, 'Churn'].mean()
hipoteses = {
    'H1 Contrato mensal':    (df.Contract == 'Month-to-month',     df.Contract != 'Month-to-month'),
    'H2 Cliente novo (≤12m)': (df.tenure <= 12,                     df.tenure > 12),
    'H3 Fibra óptica':       (df.InternetService == 'Fiber optic', df.InternetService == 'DSL'),
    'H4 Sem suporte técnico':(df.TechSupport == 'No',              df.TechSupport == 'Yes'),
    'H4 Sem segurança online':(df.OnlineSecurity == 'No',          df.OnlineSecurity == 'Yes'),
    'H5 Electronic check':   (df.PaymentMethod == 'Electronic check', df.PaymentMethod != 'Electronic check'),
}
tab = pd.DataFrame([(h, taxa(r), taxa(c)) for h, (r, c) in hipoteses.items()],
                   columns=['hipotese', 'churn_risco', 'churn_comparacao'])
tab['razao'] = tab.churn_risco / tab.churn_comparacao
tab['veredito'] = np.where(tab.razao >= 1.5, 'Confirmada', 'Não confirmada')
```

Observações sobre as escolhas:
- **H4** é verificada com "No" contra "Yes". Clientes sem internet ficam fora dessa comparação, porque não poderiam contratar o serviço.
- O corte de **H2** em 12 meses define "cliente novo" como quem está no primeiro ano. É um corte simples e fácil de explicar.

## Verificação
- `jupyter nbconvert --execute --inplace churn.ipynb` roda sem erro.
- As duas figuras existem em `figures/`.
- Esperado pelos números conhecidos do dataset: contrato mensal ≈ 43% contra ≈ 7% nos anuais, e as 5 hipóteses confirmadas. Se alguma não for confirmada, o texto registra isso. O veredito vem do dado, não da expectativa.

## Commits previstos
1. `feat(T03): adiciona taxa de churn por categoria (figura 1)`
2. `feat(T03): adiciona distribuições de tenure e mensalidade (figura 2)`
3. `feat(T03): adiciona perfil do cliente e veredito das hipóteses`
4. `docs(T03): marca T03 como concluída no backlog`

Depois dos commits: **push da milestone M1** e pausa para a sua revisão da EDA (H1).
