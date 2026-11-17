# Decision Log

Registro das decisões de escopo e de implementação do projeto **Previsão de Churn em Telecom** (C318).
Cada decisão diz o que foi decidido, quais alternativas foram consideradas e por quê.
O plano de execução que aplica essas decisões está em [`BACKLOG.md`](BACKLOG.md).

**Regra de ouro (D-00):** uma etapa só entra se for indispensável e agregar valor real ao projeto. Quando existir uma alternativa mais barata que atenda aos requisitos, ela é a escolhida. Na dúvida, a etapa fica de fora.

**Requisitos mínimos que o plano precisa cumprir** (seção 6 da proposta):
- Entregáveis da disciplina: scripts Python, documentação (proposta atualizada + README) e apresentação final.
- Checklist funcional da seção 6.2: carga, limpeza, EDA, pré-processamento, divisão estratificada, baseline + 2 modelos, tratamento do desbalanceamento, ajuste de hiperparâmetros com validação cruzada, métricas técnicas e de negócio, interpretação e simulação financeira.

Status possíveis: **Aceita** (entra no plano) · **Rejeitada** (fica fora do escopo).

---

## Estrutura e ambiente

### D-01 · Um único notebook em vez de quatro — Aceita
- **Decisão:** todo o código fica em `churn.ipynb`, com seções numeradas (dados, EDA, modelagem, impacto).
- **Alternativa rejeitada:** os quatro notebooks da seção 9 da proposta (`01_eda` … `04_impacto_negocio`).
- **Justificativa:** com quatro notebooks seria preciso salvar e recarregar dados intermediários e modelos entre eles, além de repetir imports e configuração. Um notebook só roda de ponta a ponta com um comando, e o requisito "scripts em Python / Jupyter" continua atendido.

### D-02 · Repositório plano, sem `src/`, `docs/` nem `reports/` — Aceita
- **Decisão:** estrutura final: `churn.ipynb`, `data/`, `figures/`, `slides.md`, `README.md`, `requirements.txt`, proposta, backlog e decision log, todos na raiz.
- **Alternativa rejeitada:** a árvore da seção 9 da proposta, com `src/`, `docs/`, `reports/figures/` e `data/raw/`.
- **Justificativa:** nenhuma função é reutilizada fora do notebook, então `src/` seria só indireção. Mover a proposta para `docs/` não agrega nada.

### D-03 · Dataset baixado do GitHub da IBM e versionado no repositório — Aceita
- **Decisão:** baixar `Telco-Customer-Churn.csv` pela URL raw do GitHub da IBM e fazer commit em `data/`.
- **Alternativas rejeitadas:** baixar do Kaggle (exige conta e login manual) ou documentar instruções de download.
- **Justificativa:** a URL pública funciona sem autenticação (verificado em 2026-11-16: HTTP 200, cerca de 970 KB). Com o CSV versionado, qualquer pessoa reproduz o projeto sem passo manual, e o tamanho é irrelevante para o git.

### D-04 · Dependências mínimas: pandas, scikit-learn, matplotlib e jupyter — Aceita
- **Decisão:** o `requirements.txt` lista só essas quatro bibliotecas, com as versões instaladas fixadas. O numpy vem como dependência do pandas.
- **Rejeitadas:** seaborn, xgboost/lightgbm, imbalanced-learn e shap. As justificativas estão em D-05, D-13, D-14 e D-17.
- **Justificativa:** cada dependência a mais é uma instalação e um ponto de falha. Os gráficos necessários saem direto de matplotlib/pandas.

### D-05 · Sem seaborn — Rejeitada
- **Justificativa:** os gráficos previstos (barras de taxa de churn, boxplots, curvas) são simples em matplotlib e no `DataFrame.plot` do pandas. O seaborn só deixaria os gráficos mais bonitos.

### D-06 · Sem `CLAUDE.md` — Rejeitada
- **Justificativa:** o `BACKLOG.md` já dá ao agente o contexto e as instruções de execução. Um `CLAUDE.md` repetiria esse conteúdo.

### D-07 · Sem testes unitários e sem CI; checagens via `assert` no notebook — Aceita
- **Decisão:** as checagens de controle viram `assert` logo após a carga dos dados: 7.043 linhas, 11 brancos em `TotalCharges` e taxa de churn ≈ 26,5%.
- **Alternativa rejeitada:** suíte `pytest` e GitHub Actions.
- **Justificativa:** não existe código de biblioteca para testar. Os `assert` cobrem os riscos reais, que são dado errado e limpeza errada, a custo quase zero.

---

## Limpeza e EDA

### D-08 · Brancos de `TotalCharges` preenchidos com 0 — Aceita
- **Justificativa:** os 11 casos têm `tenure = 0`, ou seja, são clientes que ainda não pagaram nada, e 0 é o valor correto. Remover as linhas também seria aceitável, mas preencher mantém todos os clientes e custa o mesmo.

### D-09 · Não padronizar `SeniorCitizen` e não fundir "No internet/phone service" com "No" — Rejeitada
- **Justificativa:** `SeniorCitizen` em 0/1 já é numérica e entra direto no modelo. O one-hot trata "No internet service" como uma categoria a mais. Nenhuma das duas mudanças altera o desempenho de forma relevante, então ficam fora.

### D-10 · Hipóteses verificadas visualmente, sem testes estatísticos formais — Aceita
- **Decisão:** cada uma das 5 hipóteses da seção 7.1 recebe um veredito (confirmada ou não) com base na taxa de churn por categoria, citando o número.
- **Alternativa rejeitada:** qui-quadrado ou teste t para cada hipótese.
- **Justificativa:** as diferenças nesse dataset são grandes (por exemplo, contrato mensal ≈ 43% contra ≈ 3% no de dois anos) e não precisam de p-valor para convencer. A proposta não exige inferência estatística.

### D-11 · Sem matriz de correlação — Rejeitada
- **Justificativa:** existem só três variáveis numéricas, e a única correlação relevante é óbvia (`TotalCharges` ≈ `tenure × MonthlyCharges`). Uma frase no notebook substitui o gráfico.
- **Nota (2026-11-16):** a matriz continua dispensável, mas a redundância de `MonthlyCharges` com os serviços contratados acabou afetando a interpretação do modelo. Ver D-30.

---

## Modelagem

### D-12 · Três modelos do scikit-learn: Regressão Logística (baseline), Árvore de Decisão e Random Forest — Aceita
- **Decisão:** atende ao requisito "baseline + pelo menos dois modelos" com os três modelos não opcionais da seção 7.3.
- **Justificativa:** os três compartilham o mesmo pipeline e o mesmo laço de treino, então o custo marginal de cada modelo é uma entrada num dicionário.

### D-13 · Sem XGBoost/LightGBM — Rejeitada
- **Justificativa:** a proposta já os trata como opcionais. Exigem outra dependência e, nesse dataset, o ganho de AUC sobre Random Forest e Regressão Logística costuma ser marginal (menos de 0,01).

### D-14 · Desbalanceamento: `class_weight='balanced'` + ajuste do limiar, sem SMOTE — Aceita
- **Decisão:** todos os modelos usam `class_weight='balanced'`, e o limiar de decisão é escolhido pela simulação financeira (D-19).
- **Alternativa rejeitada:** SMOTE com imbalanced-learn.
- **Justificativa:** SMOTE adiciona dependência, pipeline especial e uma comparação a mais. O ajuste do limiar é o que a própria proposta chama de "alavanca mais útil para o negócio" e já sai da simulação financeira.

### D-15 · Um único pré-processador para todos os modelos — Aceita
- **Decisão:** um `ColumnTransformer` com `OneHotEncoder(handle_unknown='ignore')` nas categóricas e `StandardScaler` nas numéricas, reutilizado nos três pipelines.
- **Justificativa:** o escalonamento não atrapalha as árvores, e manter um preprocessador por modelo seria código duplicado.

### D-16 · `GridSearchCV` com grades pequenas, `StratifiedKFold(5)` e métrica ROC-AUC — Aceita
- **Decisão:** de 4 a 8 combinações por modelo. As grades sugeridas estão no backlog (T04).
- **Alternativas rejeitadas:** `RandomizedSearchCV` ou grades amplas.
- **Justificativa:** o requisito pede ajuste com validação cruzada, não uma busca exaustiva. Com grades pequenas, a execução leva poucos minutos.
- **Regra de seleção:** vence o maior AUC médio na validação cruzada. Se a diferença para a Regressão Logística for menor que 0,01, fica a Regressão Logística, que é mais interpretável. O conjunto de teste é usado uma única vez, no modelo final.

### D-17 · Interpretação só do modelo final, sem SHAP — Aceita
- **Decisão:** um gráfico com as 10 variáveis mais importantes do modelo final: coeficientes, se for a Regressão Logística, ou `feature_importances_`, se for árvore.
- **Rejeitadas:** SHAP e interpretar todos os modelos.
- **Justificativa:** a direção de cada fator ("quem cancela mais") já aparece na EDA. O gráfico de importância basta para responder "por que cancelam" do ponto de vista do modelo.

### D-18 · Sem persistência do modelo, API ou deploy — Rejeitada
- **Justificativa:** não fazem parte dos entregáveis da disciplina.

---

## Impacto financeiro

### D-19 · Limiar escolhido com probabilidades out-of-fold do treino — Aceita
- **Decisão:** gerar as probabilidades do modelo final no treino com `cross_val_predict(method='predict_proba')`, escolher o limiar que maximiza o ganho e só então aplicá-lo no teste.
- **Alternativa rejeitada:** escolher o limiar direto no teste.
- **Justificativa:** escolher o limiar no teste inflaria o ganho reportado (vazamento). A correção custa uma linha e evita uma pergunta difícil na apresentação.

### D-20 · Premissas financeiras padrão, ajustáveis numa célula de parâmetros — Aceita
- **Decisão:** usar os exemplos da seção 8.2 da proposta e definir um custo fixo de contato:
  - valor do cliente retido = `MonthlyCharges × 12`
  - taxa de sucesso da retenção = 30%
  - custo por cliente contatado = US$ 50
- **Justificativa:** com valores padrão o agente não fica bloqueado esperando definição. O aluno pode mudar os três números numa célula só e reexecutar o notebook.

### D-21 · Valores mantidos em USD — Aceita
- **Justificativa:** converter para reais exigiria escolher e justificar uma taxa de câmbio sem ganho analítico. A limitação já está prevista na seção 11 da proposta.

### D-22 · Uma curva de ganho acumulado e uma tabela de cenários — Aceita
- **Decisão:** comparar três cenários (modelo, campanha aleatória com o mesmo número de contatos e não fazer nada) e calcular o indicador "contatando os X% de maior risco, alcançamos Y% dos cancelamentos".
- **Justificativa:** essa é a resposta da pergunta 4 e o principal diferencial do projeto, segundo a seção 11. Custa cerca de 20 linhas.

---

## Documentação e apresentação

### D-23 · Apresentação em Markdown (Marp), sem python-pptx nem montagem manual — Aceita
- **Decisão:** o agente escreve `slides.md` no formato Marp seguindo o roteiro da seção 12, com as figuras de `figures/` e notas do apresentador em cada slide. A exportação para PDF é feita com `npx @marp-team/marp-cli`. Se falhar, o aluno exporta pela extensão Marp do VS Code.
- **Alternativas rejeitadas:** script em python-pptx (dependência e código a mais) ou montar tudo à mão no PowerPoint (mais horas do aluno).
- **Justificativa:** texto puro é fácil para o agente gerar e para o aluno editar. O Node já está instalado (v24), então o `npx` funciona sem setup.

### D-24 · Notas do apresentador nos slides em vez de um documento separado de perguntas e respostas — Aceita
- **Justificativa:** as notas ficam onde o aluno vai ensaiar e reduzem o tempo de preparo (etapa 7 da proposta) sem criar outro artefato.

### D-25 · Proposta atualizada no lugar, apenas onde ficou desatualizada — Aceita
- **Decisão:** corrigir a taxa de churn (26,5% em todo o documento), marcar o checklist 6.2 e ajustar as seções 9 e 10 ao escopo enxuto, com link para este log.
- **Alternativa rejeitada:** reescrever a proposta ou mover o arquivo para `docs/`.
- **Justificativa:** a proposta é um entregável ("atualizado ao longo do desenvolvimento"), mas a maior parte dela continua válida. Os itens rejeitados aqui (XGBoost, SMOTE, SHAP) já estavam marcados como opcionais lá.

### D-26 · Notebook versionado com saídas — Aceita
- **Decisão:** o notebook é executado de ponta a ponta com `jupyter nbconvert --execute --inplace`, e o commit inclui as saídas.
- **Justificativa:** o professor vê os resultados no GitHub sem precisar instalar nada.

### D-27 · Declaração de uso de IA no README — Aceita
- **Justificativa:** a proposta (seção 10.2) assume esse compromisso. Basta uma seção curta no README, sem documento próprio.

---

## Processo

### D-28 · Spec mínima por tarefa, aprovada pelo aluno antes da execução — Aceita
- **Decisão:** antes de cada tarefa, o agente escreve `specs/T0x_<nome>.md` com objetivo, requisitos funcionais, entregáveis, código resumido, verificação e commits previstos, e espera a aprovação do aluno.
- **Alternativa rejeitada:** o agente executa o backlog direto, e o aluno revisa só nos pontos H1–H3.
- **Justificativa:** pedido do aluno. Cada etapa fica entendida antes de virar código, o que reduz o esforço de estudo no fim (H4) e evita retrabalho. O custo é de cerca de 10–15 min de revisão por tarefa. As specs ficam no repositório como registro do que foi feito e como.

### D-29 · Commits granulares e push a cada milestone — Aceita
- **Decisão:** um commit por mudança lógica, e push ao concluir as milestones M0–M4 definidas no backlog.
- **Alternativas rejeitadas:** um commit por tarefa sem push (plano anterior) ou push a cada commit.
- **Justificativa:** pedido do aluno. Commits pequenos deixam o histórico legível e fáceis de reverter, e o push por milestone mantém o GitHub sempre num estado coerente.

---

## Mudanças durante a execução

### D-30 · `MonthlyCharges` fora das features do modelo — Aceita
- **Contexto:** no T04, o gráfico de importância da Regressão Logística mostrou `MonthlyCharges` como a variável mais importante, com coeficiente −2,07. Isso significaria que "mensalidade alta reduz o churn", o oposto da EDA, em que quem cancela paga em média US$ 74 contra US$ 61.
- **Causa (multicolinearidade):** a mensalidade é praticamente a soma dos serviços contratados. Uma regressão de `MonthlyCharges` sobre as colunas de serviço tem R² = 0,9988. Com a mesma informação entrando duas vezes, o modelo dividiu o efeito de forma arbitrária: fibra +1,38 e streaming +0,6, compensados pela mensalidade −2,07. As previsões continuavam corretas, mas os coeficientes perdiam o significado.
- **Decisão:** retirar `MonthlyCharges` da entrada dos modelos. A coluna continua no dataset, na EDA e no cálculo do valor do cliente (T05). A informação de preço segue disponível pelas colunas de serviço e por `TotalCharges`.
- **Alternativas rejeitadas:**
  - Regularização mais forte (C = 0,1): mascara o sintoma sem resolver a causa, e a mensalidade fica com coeficiente ≈ 0.
  - Importância por permutação: também é distorcida pela colinearidade (contrato aparecia só em 4º).
  - Manter e explicar: o gráfico ficaria inutilizável na apresentação.
- **Impacto:** AUC na validação cruzada 0,8462 → 0,8459; AUC no teste 0,841 → 0,842; recall 0,783 → 0,789. Os coeficientes passaram a ser coerentes com a EDA: tempo de casa e contrato de 2 anos reduzem o risco; contrato mensal, fibra e *electronic check* aumentam.
- **📢 Para a apresentação (slide 4 ou 7):**
  - Mensagem de negócio: *"O que faz o cliente cancelar não é o preço em si, mas o que ele está pagando: fibra e streaming."*
  - Lição de modelagem: *"Prever bem não é o mesmo que explicar bem."* A primeira versão do modelo tinha AUC de 0,84, mas dizia que mensalidade alta reduz o churn. A conferência com a EDA revelou a multicolinearidade, e a correção não custou desempenho.
  - Analogia, se a banca perguntar: duas pessoas empurrando o mesmo carro sempre juntas. Dá para ver que o carro andou, mas não quanto cada uma empurrou.
