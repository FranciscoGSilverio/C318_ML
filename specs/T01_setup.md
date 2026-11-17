# Spec T01 · Setup do ambiente e dos dados
**Status:** Aguardando aprovação

## Objetivo
Deixar o projeto reproduzível: um ambiente Python isolado com as 4 dependências do projeto e o dataset dentro do repositório. Assim qualquer pessoa clona, instala e roda, sem passo manual.

## Requisitos funcionais
- RF-1: existe um ambiente virtual local (`.venv/`), fora do controle de versão.
- RF-2: `requirements.txt` lista **apenas** pandas, scikit-learn, matplotlib e jupyter, com versões fixadas (D-04).
- RF-3: o dataset original da IBM está em `data/Telco-Customer-Churn.csv`, versionado no git (D-03).
- RF-4: arquivos gerados automaticamente (`.venv/`, checkpoints do Jupyter e caches do Python) não aparecem no `git status`.

## Entregáveis
| Arquivo | Conteúdo |
|---|---|
| `.gitignore` | 3 linhas: venv, checkpoints e cache |
| `requirements.txt` | 4 linhas com `pacote==versão` |
| `data/Telco-Customer-Churn.csv` | CSV original, cerca de 970 KB, 7.043 clientes × 21 colunas |

## Como será feito (código resumido)

```bash
# 1. Ignorar o que é gerado localmente
printf '.venv/\n.ipynb_checkpoints/\n__pycache__/\n' > .gitignore

# 2. Criar o ambiente isolado e instalar só o necessário
python3 -m venv .venv
source .venv/bin/activate
pip install pandas scikit-learn matplotlib jupyter

# 3. Fixar as versões instaladas apenas desses 4 pacotes
#    (o pip freeze completo listaria cerca de 100 dependências transitivas, o que é ruído)
pip freeze | grep -iE '^(pandas|scikit-learn|matplotlib|jupyter)==' > requirements.txt

# 4. Baixar o dataset direto do GitHub da IBM (sem login, ao contrário do Kaggle)
mkdir -p data
curl -L -o data/Telco-Customer-Churn.csv \
  https://raw.githubusercontent.com/IBM/telco-customer-churn-on-icp4d/master/data/Telco-Customer-Churn.csv
```

## Verificação
```bash
python -c "import pandas, sklearn, matplotlib; print('ok')"     # → ok
python -c "import pandas as pd; print(pd.read_csv('data/Telco-Customer-Churn.csv').shape)"  # → (7043, 21)
git status --short     # → não lista nada de .venv/
```

## Commits previstos
1. `chore(T01): adiciona .gitignore e requirements.txt`
2. `data(T01): adiciona dataset Telco Customer Churn (IBM)`
3. `docs(T01): marca T01 como concluída no backlog`

Ao fim do T01 não há push. O próximo push acontece na milestone M1, depois do T03.
