# Spec T06 · Execução limpa, README e atualização da proposta
**Status:** Concluída

## Objetivo
Fechar o entregável **documentação do projeto**: um README que permite ao professor entender o resultado em 2 minutos e rodar o projeto, e a proposta original atualizada para refletir o que de fato foi feito.

## Requisitos funcionais
- RF-1: o notebook roda **de ponta a ponta sem erro** num kernel limpo e fica versionado com as saídas (D-26).
- RF-2: o `README.md` tem no máximo uma página, com:
  - objetivo e dados;
  - **resultados principais** (números iguais aos do notebook);
  - principais fatores de churn;
  - como executar;
  - estrutura do repositório;
  - limitações;
  - declaração de uso de IA (D-27);
  - links para a proposta, o decision log, o backlog e as specs.
- RF-3: a proposta é atualizada **apenas onde ficou desatualizada** (D-25):
  - taxa de churn igual a **26,5%** em todo o documento;
  - checklist 6.2 marcado;
  - seção 9 com a estrutura real do repositório;
  - nota na seção 10 dizendo que o escopo foi enxugado;
  - uma nota de revisão no topo apontando para o `DECISION_LOG.md`, com as mudanças relevantes (D-30 e D-31).

## Entregáveis
- `churn.ipynb` reexecutado.
- `README.md` novo.
- `proposta_projeto_churn_telecom.md` atualizada.

## Como será feito (resumo)

**Execução limpa:**
```bash
jupyter nbconvert --to notebook --execute --inplace churn.ipynb
```

**Esqueleto do README:**
```markdown
# Previsão de Churn em Telecom
Uma linha: o que é o projeto (C318, Inatel).

## Resultados
| Pergunta | Resposta |
| Quem vai cancelar? | AUC 0,84; alcança 82% dos churns contatando 44% da base; top 20% → 49% |
| Vale a pena agir?  | US$ 51,5 mil contra 12,1 mil da campanha aleatória (4,2×); robusto nas 9 premissas testadas |

## Principais fatores de churn      (5 bullets, da EDA e do modelo)
## Como executar                     (venv, pip install -r, abrir o notebook)
## Estrutura do repositório          (árvore curta)
## Limitações                        (premissas, dataset fictício, momento único)
## Uso de IA                         (ver pergunta abaixo)
## Documentação                      (proposta, DECISION_LOG, BACKLOG, specs/)
```

**Alterações na proposta:**

| Onde | Antes | Depois |
|---|---|---|
| Topo | — | Nota: "Escopo revisado durante a execução: ver `DECISION_LOG.md` (destaques: D-30 e D-31)" |
| Seção 4 | "cerca de 26% de churn" | "26,5% de churn" |
| Seção 5.4 | "cerca de 73% No e 27% Yes" | "73,5% No e 26,5% Yes" |
| Seção 6.2 | `- [ ]` | `- [x]` |
| Seção 7.2 | — | 1 bullet: `MonthlyCharges` fica fora das features do modelo (D-30) |
| Seção 8.1 | "acerta cerca de 73%" | "acerta 73,5%" |
| Seção 9 | Árvore com 4 notebooks, `src/` e `docs/` | Estrutura real (1 notebook, `specs/`, `figures/`) |
| Seção 10 | — | Nota: escopo reduzido conforme o `DECISION_LOG.md` |

As seções 1 a 3, 5.1 a 5.3, 11, 12 e 13 continuam válidas e não mudam.

## Declaração de uso de IA
Decidido pelo aluno: a declaração **cita o Claude Code nominalmente**, consistente com a seção 10 da proposta.

## Verificação
- O `nbconvert` termina sem erro.
- Todos os números do README conferem com as saídas do notebook (checagem manual, item a item).
- Os links do README apontam para arquivos que existem.

## Commits previstos
1. `chore(T06): reexecuta o notebook de ponta a ponta` (só se houver mudança nas saídas)
2. `docs(T06): adiciona README do projeto`
3. `docs(T06): atualiza proposta com taxa de churn, checklist e escopo revisado`
4. `docs(T06): marca T06 como concluída no backlog`

Depois dos commits: **push da milestone M3**.
