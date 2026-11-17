# Previsão de Churn em Telecom

Projeto individual da disciplina **C318 – Fundamentos de Machine Learning** (Inatel). Um modelo de classificação estima o risco de cancelamento de cada cliente de uma operadora de telecomunicações. Com ele, o projeto mede quanto uma campanha de retenção guiada pelo modelo preservaria de receita.

**Dados:** [Telco Customer Churn](https://github.com/IBM/telco-customer-churn-on-icp4d) (IBM), com 7.043 clientes, 21 colunas e 26,5% de churn. É uma operadora fictícia da Califórnia.

## Resultados

| Pergunta de negócio | Resposta |
|---|---|
| **Quem vai cancelar?** | Regressão Logística com AUC de 0,84 no teste. Contatando 44% da base, alcança **82% dos churns**. Só os **20% de maior risco já concentram 49%** deles (2,5× o acaso). |
| **Por que cancelam?** | Contrato mensal, primeiro ano de casa, fibra óptica, pagamento por *electronic check* e ausência de segurança online ou suporte técnico. |
| **Quem é o cliente de risco?** | Contrato mensal, até 12 meses de casa, fibra, mensalidade acima da média e sem serviços de proteção. |
| **Vale a pena agir?** | **Sim.** No teste, a campanha guiada pelo modelo gera **US$ 51,5 mil**, contra US$ 12,1 mil de uma campanha aleatória do mesmo tamanho (**4,2×**) e US$ 27,5 mil de contatar todos. O modelo vence nas 9 combinações de premissas testadas. |

As premissas financeiras são: valor do cliente retido = 12 mensalidades, 30% de sucesso na retenção e US$ 50 por contato. Elas ficam numa célula de parâmetros do notebook e podem ser ajustadas.

### Achados de modelagem
- **Random Forest e Regressão Logística empatam** (AUC de 0,847 e 0,846 na validação cruzada). Ficou a Regressão Logística, por ser interpretável.
- **`MonthlyCharges` foi retirada do modelo.** A mensalidade é a soma dos serviços contratados (R² = 0,9988), e mantê-la invertia o sinal dos coeficientes (multicolinearidade). O desempenho não mudou. Ver [D-30](DECISION_LOG.md).
- **O limiar de decisão importa menos do que o esperado.** A curva de ganho é plana entre 0,35 e 0,55, e o valor do modelo está em **ordenar bem os clientes por risco**. O limiar só faz diferença quando as premissas mudam.

## Como executar

Requer Python 3.10+.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook churn.ipynb
# ou, sem abrir o navegador:
jupyter nbconvert --to notebook --execute --inplace churn.ipynb
```

O dataset já está no repositório, e o notebook roda de ponta a ponta em menos de 1 minuto.

## Estrutura

```
├── churn.ipynb                        # notebook único: dados, EDA, modelagem, impacto e conclusões
├── data/Telco-Customer-Churn.csv      # dataset original (IBM)
├── figures/                           # figuras geradas pelo notebook
├── proposta_projeto_churn_telecom.md  # proposta do projeto
├── BACKLOG.md                         # plano de execução por tarefas
├── DECISION_LOG.md                    # decisões de escopo e de modelagem, com justificativas
├── specs/                             # spec de cada tarefa, aprovada antes da execução
└── requirements.txt
```

## Limitações
- **Premissas financeiras hipotéticas.** A análise de sensibilidade mostra que a conclusão se mantém, mas os valores em US$ dependem delas.
- **Modelo financeiro simplificado.** A taxa de sucesso é igual para todos, e não se modela o custo do desconto para quem não ia cancelar nem o risco de o contato estimular um cancelamento.
- **Dados fictícios, de outro país e de um único momento.** Em produção, o modelo precisaria ser retreinado periodicamente com dados reais.
- **Escala.** Os valores em US$ são do conjunto de teste (1.409 clientes, 20% da base).

## Uso de IA

O projeto foi desenvolvido com apoio do **Claude Code**, assistente de programação agêntico. O assistente escreveu e executou a maior parte do código e redigiu a documentação.

O autor fez o restante:
- definiu o escopo;
- revisou e aprovou a spec de cada tarefa antes da execução ([`specs/`](specs/));
- tomou as decisões de modelagem e de negócio ([`DECISION_LOG.md`](DECISION_LOG.md));
- revisou os resultados.

## Documentação
- [Proposta do projeto](proposta_projeto_churn_telecom.md)
- [Decision log](DECISION_LOG.md): por que cada etapa foi incluída ou rejeitada
- [Backlog](BACKLOG.md): plano de execução e estado das tarefas
- [Specs](specs/): o que cada tarefa entregou e como
