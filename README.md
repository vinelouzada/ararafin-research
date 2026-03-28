# AraraFin — Análise de Sentimentos em Notícias do Mercado Financeiro Brasileiro

Este repositório contém os artefatos de uma pesquisa acadêmica sobre a aplicação de **Modelos de Linguagem de Grande Escala (LLMs)** na análise de sentimentos de notícias do mercado financeiro brasileiro. O objetivo é avaliar se LLMs são capazes de classificar corretamente o impacto de notícias financeiras para diferentes tipos de investimento: **Ações**, **Fundos de Investimento Imobiliário (FIIs)** e **Renda Fixa** — em comparação a um gabarito elaborado por especialistas humanos.

---

## Estrutura do Repositório

```
ararafin-research/
├── prompts.md
├── analise_sentimentos_textos_financeiros_gabarito.csv
├── ararafin_llm_avaliacao.csv
├── Notícias do Mercado Financeiro - Sentimentos - Engenheiro I.csv
├── Notícias do Mercado Financeiro - Sentimentos - Engenheiro R.csv
├── Notícias do Mercado Financeiro - Sentimentos - Engenheiro V.csv
├── Notícias do Mercado Financeiro - Sentimentos - Investidor R.csv
├── Análises_de_desempenho_dos_LLMs.ipynb
└── Consolidação_do_Dataset_de_Referência_notícias_rotuladas_do_mercado_financeiro_brasileiro.ipynb
```

---

## Descrição dos Arquivos

### `prompts.md`

Documenta os **prompts de sistema** utilizados para instruir os LLMs na tarefa de classificação de sentimentos. Há um prompt distinto para cada tipo de investimento avaliado:

- **FIIs** — orienta o modelo como consultor especializado em Fundos de Investimento Imobiliário, com critérios como Dividend Yield, vacância, IFIX e P/VP.
- **Ações** — orienta o modelo como consultor do mercado de ações da B3, com critérios como ROI, liquidez e cotação do ativo.
- **Renda Fixa** — orienta o modelo como consultor de Renda Fixa, com critérios como rentabilidade, indexador (CDI, IPCA, Prefixado) e risco de crédito.

Todos os prompts solicitam uma resposta no formato JSON com os campos `sentiment` (POSITIVO, NEGATIVO ou NEUTRO) e `justify`.

---

### `analise_sentimentos_textos_financeiros_gabarito.csv`

Dataset de referência (**gabarito**) contendo notícias do mercado financeiro brasileiro anotadas com o sentimento esperado para cada tipo de investimento. É o conjunto utilizado como ground truth para avaliar o desempenho dos modelos.

**Colunas principais:**

| Coluna | Descrição |
|---|---|
| `_id` | Identificador único da notícia |
| `title` | Título da notícia |
| `body` | Corpo completo da notícia |
| `summary` | Resumo gerado da notícia |
| `published_at` | Data de publicação |
| `source` | Fonte da notícia |
| `url` | URL de origem |
| `Sentimento esperado para Renda Fixa` | Classificação esperada (POSITIVO / NEGATIVO / NEUTRO) |
| `Sentimento esperado para FIIs` | Classificação esperada (POSITIVO / NEGATIVO / NEUTRO) |
| `Sentimento esperado para Ações` | Classificação esperada (POSITIVO / NEGATIVO / NEUTRO) |

---

### `ararafin_llm_avaliacao.csv`

Contém as **classificações geradas pelos LLMs** para as notícias do dataset. Cada linha representa a resposta de um modelo para uma notícia e tipo de investimento específico.

**Colunas principais:**

| Coluna | Descrição |
|---|---|
| `id` | Identificador da notícia |
| `text` | Texto da notícia submetido ao modelo |
| `classification` | Classificação retornada pelo modelo (POSITIVE / NEGATIVE / NEUTRAL) |
| `model_ai` | Nome do modelo utilizado |
| `prompt` | Prompt utilizado na chamada |
| `investment_target` | Tipo de investimento alvo (FIIs, Ações ou Renda Fixa) |
| `created_at` / `updated_at` | Timestamps da avaliação |

---

### Arquivos de Anotação por Especialistas

Estes arquivos contêm as classificações realizadas manualmente por avaliadores humanos utilizados para compor o gabarito e calcular o índice de concordância entre anotadores.

| Arquivo | Perfil do Anotador |
|---|---|
| `Notícias do Mercado Financeiro - Sentimentos - Engenheiro I.csv` | Engenheiro com perfil técnico (anotador I) |
| `Notícias do Mercado Financeiro - Sentimentos - Engenheiro R.csv` | Engenheiro com perfil técnico (anotador R) |
| `Notícias do Mercado Financeiro - Sentimentos - Engenheiro V.csv` | Engenheiro com perfil técnico (anotador V) |
| `Notícias do Mercado Financeiro - Sentimentos - Investidor R.csv` | Investidor com perfil financeiro (anotador R) |

Todos os arquivos compartilham a mesma estrutura de colunas do gabarito, com campos de sentimento esperado para cada classe de investimento.

---

### `Consolidação_do_Dataset_de_Referência_notícias_rotuladas_do_mercado_financeiro_brasileiro.ipynb`

Notebook Jupyter responsável pela **construção e validação do dataset de referência**. Realiza as seguintes etapas:

1. Importação das classificações individuais de cada anotador.
2. Cálculo do **Kappa de Fleiss** para medir o grau de concordância entre os anotadores.
3. Consolidação das anotações em um único gabarito para ser utilizado como ground truth nas avaliações dos LLMs.

---

### `Análises_de_desempenho_dos_LLMs.ipynb`

Notebook Jupyter com as **análises de desempenho dos modelos de linguagem** avaliados na pesquisa. Compara as classificações geradas pelos LLMs com o gabarito humano, calculando métricas como:

- **Acurácia**
- **Precisão**
- **F1-Score**

Os resultados são apresentados com visualizações gráficas por modelo e por tipo de investimento.

---

## Classes de Sentimento

Cada notícia é classificada para cada tipo de investimento com uma das três categorias:

| Classe | Significado |
|---|---|
| **POSITIVO** | A notícia tende a valorizar o ativo ou setor |
| **NEGATIVO** | A notícia tende a desvalorizar o ativo ou setor |
| **NEUTRO** | Impacto irrelevante ou equilibrado |

---

## Requisitos

Os notebooks utilizam as seguintes bibliotecas Python:

```
pandas
scikit-learn
statsmodels
matplotlib
requests
```
