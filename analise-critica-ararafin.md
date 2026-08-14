# Análise crítica — AraraFin (revisão para nova submissão)

> Documento de trabalho. Reúne (1) o entendimento do artigo, (2) a análise dos comentários dos revisores, (3) problemas independentes que os revisores não encontraram, (4) a decisão sobre os modelos, (5) o plano de ação priorizado e (6) a decisão estratégica. Ao final, um **checklist ordenado por melhor relação esforço × impacto** (fazer de cima para baixo).
>
> **Base factual:** além de ler o artigo, os prompts e a bibliografia, as métricas foram **recomputadas a partir dos CSVs originais** (`analise_sentimentos_textos_financeiros_gabarito.csv` e `ararafin_llm_avaliacao.csv`). Vários achados abaixo vêm dessa recomputação.

---

## ⚠️ Achado que atravessa toda a análise

**O "F1-score" e a "Precisão" reportados na Tabela 4 são médias *ponderadas* (weighted), não macro — apesar de o texto do artigo afirmar que a precisão é a "média aritmética das precisões individuais de cada classe" (ou seja, macro).**

Verificação por recomputação (casam com weighted, não com macro):

| Célula (Tab. 4) | Reportado | macro | weighted |
|---|---|---|---|
| Precisão RF, Sabiazinho 3 | 0,84 | **0,385** | **0,844** ✅ |
| Precisão RF, GPT-4o | 0,87 | 0,464 | **0,867** ✅ |
| F1 RF, Sabiazinho 3 | 0,84 | **0,388** | **0,836** ✅ |
| F1 FIIs, Sabiazinho 3 | 0,81 | 0,680 | **0,810** ✅ |
| F1 Ações, Sabiazinho 3 | 0,54 | 0,558 | **0,544** ✅ |

Em Renda Fixa (90% neutro) isso significa que a tabela reporta essencialmente o desempenho na classe neutra e **esconde que o modelo praticamente não detecta as classes positiva/negativa**. Este é o eixo do problema mais grave do artigo e precisa ser corrigido **antes** que um revisor recompute.

---

## 1. Entendimento do trabalho

- **Problema de pesquisa.** Análise de sentimentos de notícias financeiras em pt-BR é mal atendida: a ferramenta de referência (FinBERT-PT-BR) faz sentimento *genérico* e não distingue que a mesma notícia impacta Renda Fixa, FIIs e Ações de formas diferentes.
- **Motivação.** Uma mesma notícia (ex.: Selic sobe) é positiva para Renda Fixa e negativa para Ações/FIIs. Um classificador genérico não produz três leituras distintas.
- **Contribuição declarada (tríplice):** (1) a ferramenta AraraFin (web) com classificação sensível ao domínio via prompts especializados (role prompting + one-shot JSON + temperatura 0); (2) estudo comparativo de 5 LLMs considerando desempenho **e** custo/tempo; (3) dataset de 628 notícias rotuladas por domínio (disponibilizado).
- **Pergunta de pesquisa (implícita, nunca formalizada):** qual LLM oferece a melhor relação desempenho/custo para a tarefa, e a abordagem LLM+prompt supera o FinBERT-PT-BR?
- **Metodologia / dataset.** Web scraping de 4 veículos (InfoMoney, FIIs Notícias, MoneyTimes, CNN Brasil), 24 dias (12/08–05/09/2025), 780 → 628 textos após limpeza. Cada texto recebeu um **resumo gerado por LLM**; anotadores humanos **e** modelos usaram o **resumo**, não o texto original. Quatro anotadores (3 engenheiros + 1 profissional de mercado), rótulo por moda, empate desfeito pelo especialista. Concordância por Fleiss' Kappa (FIIs 0,56; Ações 0,57; RF 0,35). Correlação de Pearson entre domínios. 628 × 3 domínios = 1.884 classificações por modelo.
- **Modelos (confirmados nos dados, 1.884 linhas cada):** `gpt-5.1-2025-11-13`, `gpt-5-mini-2025-08-07`, `gpt-4o-2024-08-06`, `sabiá-3.1`, `sabiazinho-3` + FinBERT-PT-BR como baseline.
- **Métricas.** Acurácia, Precisão, F1 (média mal especificada — ver achado acima).
- **Resultados declarados.** GPT-4o melhor global; Sabiazinho 3 "competitivo" com custo ~14–18× menor e mais rápido → escolhido para produção; AraraFin "supera" FinBERT em todas as modalidades (">100% em FIIs e RF, ~12% em Ações").
- **Limitações declaradas.** Ações baixo; RF precisa de melhor guia de anotação; prompts a evoluir. Pouco além disso.

---

## 2. Análise dos comentários dos revisores

Severidade: **[C]** crítico · **[I]** importante · **[Mo]** moderado · **[Me]** menor · **[Op]** opinião.

### Temas de consenso (2+ revisores) — os decisivos

#### T1. Comparação com FinBERT-PT-BR é injusta — [C] — os 3 revisores
- **Pedido:** FinBERT faz sentimento *geral* e não recebe o domínio; o gabarito é rótulo de *impacto por domínio*. Não resolvem a mesma tarefa; não se pode afirmar "AraraFin supera FinBERT".
- **Concordo (totalmente).** É a crítica mais sólida. FinBERT é estruturalmente incapaz de produzir 3 respostas distintas para o mesmo texto; está sendo pontuado contra rótulos que não consegue gerar.
- **Lacuna real?** Sim, mas de **enquadramento (claim)**, não de dados. O experimento é válido; a interpretação é exagerada.
- **Corrige com texto?** ~80% sim: reposicionar o FinBERT como **referência agnóstica de domínio / limite inferior** que demonstra a insuficiência da abordagem genérica — não como competidor head-to-head. Remover "supera/outperformed".
- **Novo experimento?** Só para blindagem máxima (adaptar FinBERT: 3 classificadores por domínio, ou concatenar domínio, ou tarefa de sentimento *geral* com entrada/saída idênticas). **Escopo grande e desnecessário para a contribuição original — não fazer.**
- **Menor alteração:** reescrever §5.2 + abstract + conclusão + parágrafo explícito de ameaça à validade mantendo a Tabela 5.

#### T2. Desbalanceamento + acurácia enganosa + baseline majoritário — [C] — revisores 2 e 3
- **Pedido:** com 90% neutro em RF, um classificador "tudo-neutro" tem 90% de acurácia — acima dos 0,83 do modelo. Não caracterizar 0,83 como "satisfatório" sem mostrar desempenho nas classes positiva/negativa.
- **Concordo, e os dados confirmam de forma pior do que os revisores perceberam.**

Baselines "tudo-neutro" (recomputados) vs. melhor acurácia do artigo:

| Domínio | Baseline tudo-neutro (acc) | Melhor acc no artigo | Veredito |
|---|---|---|---|
| **Renda Fixa** | **0,900** | 0,84 (GPT-4o) / 0,83 (Sab) | **ABAIXO do trivial** |
| FIIs | 0,650 | 0,83 (Sab) | acima (+18pp) ✔ |
| Ações | 0,457 | 0,59 (GPT-4o) | acima (+13pp) ✔ |

Macro-F1 real (o artigo **não** reporta) — baseline macro-F1 tudo-neutro: RF 0,316 / FIIs 0,263 / Ações 0,209:

| Modelo | RF macro-F1 | FIIs macro-F1 | Ações macro-F1 |
|---|---|---|---|
| Sabiazinho 3 | **0,388** (F1 negativo = **0,05**) | 0,680 | 0,558 |
| GPT-4o | 0,468 | 0,721 | 0,607 |

- **Conclusão factual:** o "excelente desempenho em Renda Fixa" **não se sustenta**. Em RF o modelo quase não detecta negativas (recall 0,04 = 1 em 25) nem positivas (recall 0,26). O 0,83/0,84 é quase inteiramente a classe neutra. **É o ponto que mais ameaça a aprovação.**
- **Discordo parcialmente do revisor 2:** ele juntou "renda fixa e FIIs" sob o baseline de 90%. Isso está **errado para FIIs** (65% neutro; Sabiazinho faz 0,83 acc e macro-F1 0,68, batendo o trivial com folga). **FIIs é o resultado genuíno e defensável — usar isso a seu favor na resposta.**
- **Corrige com texto/análise?** Sim, sem novos experimentos: reportar macro-F1, F1 por classe, matriz de confusão e baseline majoritário (tudo dos CSVs existentes) e reescrever as conclusões de RF.

#### T3. Resumo por LLM: fidelidade e viés — [I] — revisores 1, 2 e 3
- **Pedido:** humanos e modelos usaram resumos gerados por LLM. Pode remover informação, introduzir viés de sumarização e **favorecer a família que gerou o resumo**. Pedem validação da fidelidade ou uso do texto original.
- **Concordo parcialmente.** Risco real de vantagem se o resumo foi gerado por um dos modelos avaliados. **Confirmado: o artigo não informa qual modelo gerou os resumos e essa informação não está nos notebooks** (também é falha de reprodutibilidade — T4).
- **Corrige com texto?** Parcialmente. Mínimo: (a) informar modelo/prompt/limite do resumo; (b) validação leve — amostra de ~30–50 resumos, checagem manual de fidelidade + reanotar a partir do texto original e medir taxa de mudança de rótulo. Barato, não é novo projeto.
- **Novo experimento?** Mini-estudo de concordância resumo-vs-original, sim. Refazer o dataset com texto integral, **não**.

#### T4. Reprodutibilidade — modelo dos resumos e detalhes ausentes — [I] — revisores 2 e 3
- **Pedido:** qual modelo gerou os resumos, prompt, limite de tamanho, houve validação.
- **Concordo.** Confirmado ausente. Correção puramente textual (você tem a informação). **Esforço baixo, impacto alto.**

#### T5. Figuras 4 e 5 idênticas — [Me] — revisores 1 e 3
- **Confirmado no `main.tex`:** linhas 669 e 676 incluem o mesmo `tempo_medio_por_noticia.pdf` duas vezes. Uma deveria ser `tempo_total_para_1884.pdf`. Corrigir sem falta (dá impressão de descuido).

#### T6. Fonte da Tabela 1 (e outras) pequena — [Me] — revisores 1 e 2
- Cosmético. `\resizebox` encolhe demais. Ajustar.

### Revisor 1 — pontos exclusivos
- **Dataset pequeno/estreito (628, 24 dias) + coletar notícias específicas por ativo — [I / Op].** Concordo parcialmente. A limitação temporal é real e deve ser **declarada** (texto). Mas coletar corpora separados por ativo **contradiz a tese** (o ponto é a *mesma* notícia gerar leituras diferentes; corpora distintos destroem a demonstração e a correlação de Pearson). Aqui o revisor propõe **outro trabalho** — defender o desenho, não atender.
- **Custo-benefício incompleto (latência confundida com capacidade; specs locais irrelevantes para API) — [I].** Concordo. Separar (i) custo de inferência (tokens×preço, objetivo), (ii) latência de API (não controlada), (iii) throughput. Remover o MacBook M2 como se explicasse desempenho de modelos em nuvem (só faz sentido para o FinBERT local). Texto/análise.
- **"GPT-4o é o melhor modelo para a maioria das tarefas" é polêmico — [Me].** Concordo (marketing citado como fato). Reescrever neutro.
- **Descrever melhor os prompts / critério do exemplo one-shot — [Mo].** Concordo. Trazer ao corpo/apêndice e explicar a escolha do one-shot. Texto.
- **"BECAUSE it is a smaller model" — [Me].** Erro lógico; era "even though/embora". Corrigir.

### Revisor 2 — pontos exclusivos
- **Originalidade fraca ("apenas atualiza modelos") — [C/Op].** Concordo parcialmente — perigoso. Defesa honesta: a novidade é (a) a formulação da tarefa como **classificação de impacto sensível ao domínio** (1 notícia → 3 rótulos condicionados), (b) o **dataset multi-domínio inédito**, (c) a evidência de que prompts segmentados capturam essas diferenças. **Mas hoje (c) está sub-demonstrada** (falta ablação). Corrige-se reposicionando a narrativa **e** com a ablação mínima.
- **Treinamento/guia de anotação, divergência vs especialista, exemplo "Selic 15%" — [I].** Concordo. Mínimo: (i) disponibilizar/descrever o guia de anotação; (ii) reportar % de unanimidade / maioria / empate (computável dos 4 CSVs de anotação). Baixo-médio, sem experimento.
- **Notícias de sinal misto (queda de lucro + alta de receita) — [Mo].** Concordo. Um parágrafo explicando a regra (NEUTRO = efeitos se anulam; moda entre anotadores). Texto.
- **Explicação de Ações genérica/"informação nula" — [Mo].** Concordo (hand-waving). Pede análise de erros. **Você tem os dados** — matriz de confusão + 15–20 exemplos errados. Nota: Ações tem **macro-F1 (0,56–0,61) melhor que Renda Fixa (0,39–0,47)** — o artigo inverte a narrativa. Médio, sem experimento.
- **Referências incompletas — [Me].** Confirmado no `.bib`: `beltramini2024analise` (sem veículo/tipo), `darwish2025stockforecasting` (autor "M. and others"), `yan2023prediccao` presente mas aparentemente não citado. Padronizar.

### Revisor 3 — pontos exclusivos (o mais técnico)
- **Ablação dos prompts — [C].** Obrigatório. O artigo atribui o desempenho à segmentação + role + one-shot, mas todos os modelos usam sempre o prompt completo — não dá para saber se o ganho vem da segmentação ou da capacidade geral do LLM. **Concordo integralmente.**
  - **Menor experimento suficiente:** ablação em **um único modelo** (o de produção): (i) prompt genérico de sentimento vs (ii) prompt com domínio + role + one-shot completo, no mesmo dataset. ~1.884 chamadas extras por condição (~35 min, custo trivial). Opcional 3ª condição (domínio sem role/one-shot). **Não fazer a grade completa × 5 modelos — isso seria escopo excessivo.**
  - É a **única evidência causal** possível para o claim-título ("domain-sensitive"). Sem ela, o revisor 2 tem razão.
- **Tipo de média não esclarecido; pedir por-classe + macro + weighted + matriz de confusão — [C].** Concordo — e o artigo está **factualmente errado** (descreve macro, reporta weighted). Corrigir é obrigatório; resolve T2. Sem experimento.
- **Evitar "ganhos >100%"; usar pontos percentuais — [I].** Concordo. 0,17→0,83 é +66pp; o "100%+" infla porque o baseline é patologicamente baixo. Trocar por pp. Texto.
- **Qualificação dos 3 anotadores — [Mo].** Concordo (sobrepõe rev. 2). Descrever perfil/treinamento. Texto.
- **Fleiss κ=0,35 em RF ↔ conclusões mais fortes onde a incerteza é maior — [I].** Concordo, inconsistência real. Resolve junto com T2 (rebaixar RF) + reportar unanimidade/maioria/empate.
- **Custo-benefício: tokens médios in/out, custo total, custo por notícia, projeções mensais — [Mo].** Concordo. Fortalece a contribuição (2) com dado objetivo; você tem os dados. Baixo-médio, sem experimento. Substitui a métrica de latência frágil.
- **"Transparência" via justificativas ≠ explicação fiel — [Mo].** Concordo. Trocar por "fornecimento de justificativas textuais"; opcional avaliar amostra de justificativas (P2).
- **Revisão de apresentação (padronizar "AraraFin", tirar linguagem promocional, completar refs) — [Me].** Concordo. Texto.

---

## 3. Problemas que os revisores NÃO encontraram (revisão independente)

1. **[C] Inconsistência factual macro vs weighted** (detalhada no topo). O revisor 3 chegou perto (ambiguidade), mas ninguém viu que o texto **afirma macro e reporta weighted**. Motivo de rejeição se um revisor recomputar. Corrigir antes.
2. **[I] A conclusão prática (adotar Sabiazinho 3) é parcialmente artefato da métrica errada.** Em **macro-F1, GPT-4o supera o Sabiazinho em todos os domínios** (RF 0,47 vs 0,39; FIIs 0,72 vs 0,68; Ações 0,61 vs 0,56). A escolha do Sabiazinho ainda é defensável **por custo** (~90% do macro-F1 do GPT-4o a ~7% do custo), mas precisa ser reescrita nesses termos, não como "qualidade equivalente".
3. **[I] A narrativa RF-bom / Ações-ruim está invertida em macro-F1.** O artigo celebra RF (macro-F1 0,39, quase trivial) e lamenta Ações (macro-F1 0,56, o mais útil). Sob a métrica correta, **Ações é onde a abordagem mais agrega sobre o baseline**.
4. **[I] Ausência de significância estatística / IC.** Diferenças como 0,82 vs 0,83 (FIIs) são tratadas como reais sem IC/bootstrap (n=628). Bootstrap de IC 95% no macro-F1 é barato e blinda as comparações.
5. **[Mo] Sem RQ formal nem mapeamento objetivo→resultado→conclusão.** Facilita a acusação de "falta de foco/originalidade". Enunciar 2–3 RQs explícitas organiza tudo e é barato.
6. **[Mo] Possível indução determinística no prompt.** Regras como "SELIC: aumento é POSITIVO" (RF) / "NEGATIVO" (Ações/FIIs) embutem conhecimento à mão — parte do "acerto" é regra codificada, não capacidade do modelo. Reforça a necessidade da ablação e deve ser declarado (desempenho é da dupla prompt+modelo).
7. **[Mo] Contaminação temporal.** Notícias ago–set/2025 vs modelos como GPT-5.1 (nov/2025) podem ter os eventos no treinamento. Uma frase de ameaça à validade.
8. **[Me] "Acurácia >100%"** no abstract/conclusão é impossível em termos absolutos (é ganho relativo). Junta-se ao ponto de pp.
9. **[Me] Seção da ferramenta (tecnologias/UI) está comentada no LaTeX.** A contribuição (1) "a ferramenta" quase some; hoje o artigo é ~90% benchmark. Há descompasso entre o título ("A LLM-Based Solution") e o conteúdo. Decidir: mostrar a ferramenta ou ajustar título/claims.

**Adequado (não inventar correção):** a ideia central (impacto sensível a domínio) é original o suficiente; Fleiss' Kappa é apropriado; a correlação de Pearson é um acréscimo correto e elegante; a coleta multi-fonte com rastreabilidade é sólida; temperatura 0 + one-shot JSON são escolhas defensáveis; o dataset é uma contribuição legítima.

---

## 4. Decisão sobre os modelos (Sabiazinho 3 será descontinuado)

**Contexto atualizado:** o modelo descontinuado é o **Sabiazinho 3** — o modelo escolhido para produção e um dos melhores resultados (melhor acurácia em FIIs, mais barato e mais rápido). Ou seja, não é um ponto de comparação qualquer que sai: é o **modelo de produção recomendado** que deixa de existir. O sucessor é o **Sabiazinho 4**.

| Opção | Impacto científico | Escopo/esforço | Risco de inconsistência | Comparabilidade | Risco de "mexer após ver o resultado" |
|---|---|---|---|---|---|
| **A** – manter tudo, só explicar que Sabiazinho 3 foi descontinuado | Neutro cientificamente, mas **a recomendação de produção aponta para um modelo morto** | Nenhum (texto) | Nenhum | Total | Nenhum |
| **B** – manter tudo + adicionar Sabiazinho 4 | **Positivo** (atualiza e dá recomendação de produção viável) | Baixo (1× pipeline, ~35 min) | Baixo se nas mesmas condições | Total (mesmo dataset/prompt) | **Baixo** (há motivo externo legítimo) |
| **C** – substituir Sabiazinho 3 pelo Sabiazinho 4 | Perde o data point vencedor em custo-benefício | Médio | Médio | Perde comparabilidade histórica | Médio (mas justificável) |
| **D** – refazer todo o conjunto | Alto custo, baixo retorno | Alto | Alto | Zero | Alto |

**Recomendação: Opção B (manter os 5 + adicionar Sabiazinho 4).**

- **Descontinuação ≠ invalidação.** O experimento foi rodado em data carimbada, com resultados arquivados nos CSVs — continua válido. Basta a nota: "Sabiazinho 3 (`sabiazinho-3`) foi avaliado em [data]; posteriormente entrou em depreciação, o que motivou a avaliação do sucessor Sabiazinho 4 como modelo de produção."
- **Por que a adição sobe de prioridade:** sem o Sabiazinho 4, a conclusão prática ("adotar Sabiazinho 3") recomenda um modelo indisponível. Com o Sab3 saindo, avaliar o sucessor é **necessário para a validade da contribuição (2)**, não um extra.
- **Risco de cherry-picking agora é baixo:** há um motivo externo (o modelo saiu). Manter o Sabiazinho 3 na tabela + adicionar o 4 é transparente.
- **Não substituir (C):** perder o data point do Sabiazinho 3 empobrece a análise de custo-benefício (era o melhor nesse eixo). Melhor manter os dois.
- **Conjunto mínimo suficiente para a RQ:** (i) um LLM de fronteira forte [GPT-4o e/ou GPT-5.1]; (ii) um LLM barato especialista em pt-BR [Sabiazinho 3 → 4]; (iii) o baseline [FinBERT-PT-BR]. Os 5 atuais já cobrem; **6 com o Sabiazinho 4 é o teto razoável.** Adicionar uma frota (Claude, Gemini, Llama-pt) seria escopo sem retorno.
- **Condição para a Opção B ser limpa:** rodar o Sabiazinho 4 em **condições idênticas** (mesmo dataset, mesmos prompts, temperatura 0, mesma pipeline) e reportar **mesmo que seja pior**.

> Adicionar/trocar modelos **não resolve** o problema central (macro-F1/baseline/FinBERT). Priorize a métrica antes do modelo.

---

## 5. Decisão estratégica (respostas objetivas)

1. **Contribuição científica defensável?** Sim, mas hoje mal apresentada. A contribuição sólida é a **formulação da tarefa de sentimento-de-impacto sensível a domínio + o dataset multi-domínio + a evidência (a completar com ablação) de que FIIs e Ações se beneficiam disso**. Não é "supera FinBERT" nem "excelente em RF".
2. **3 maiores riscos:** (a) revisor recomputar e ver **RF abaixo do baseline trivial** e weighted-como-macro; (b) comparação injusta com FinBERT lida como claim insustentável; (c) baixa originalidade prosperar por falta de **ablação**.
3. **Críticas que você DEVE atender:** T1 (FinBERT), T2 (desbalanceamento/métricas), T4 (reprodutibilidade dos resumos), ablação (rev. 3), figuras/refs.
4. **Resolvíveis só com texto/reanálise (sem novo experimento):** métricas por classe/macro/matriz de confusão, reenquadramento do FinBERT, "ganhos em pp", "transparência", análise de erros de Ações, custo por tokens, unanimidade/maioria/empate, qualificação de anotadores, sinal misto, latência, refs, figuras.
5. **Essencialmente um novo trabalho (não atender agora):** corpora por ativo (rev. 1); re-treinar/adaptar FinBERT multitarefa e avaliação de sentimento geral com novo gabarito (rev. 3); refazer o dataset com textos integrais.
6. **Adicionar Sabiazinho 4?** **Sim** — e agora com prioridade elevada, porque o Sabiazinho 3 (modelo de produção) foi descontinuado. Condições idênticas, reporte honesto.
7. **Substituir ou manter o descontinuado?** **Manter o Sabiazinho 3** na tabela e **adicionar** o Sabiazinho 4 (Opção B). Substituir empobrece a análise de custo-benefício.
8. **Quantos modelos no total?** **5 a 6 LLMs + FinBERT.** 6 com o Sabiazinho 4. Não passar disso.
9. **Refazer experimentos antigos ou complementar?** **Complementar, não refazer.** As classificações existentes são válidas. Único experimento novo *necessário*: a **ablação mínima de prompt em 1 modelo**; os demais novos são pequenos (validação de resumo, Sabiazinho 4).
10. **Estratégia de orientação (maximizar aprovação sem inflar escopo):** o gargalo não é dados — é **rigor de métrica e enquadramento**. Sprint 1 = reanálise (zero experimento novo) neutraliza ~70% das críticas; Sprint 2 = 1 experimento mínimo (ablação) + 1 mini-estudo (fidelidade do resumo) + Sabiazinho 4; reposicionar a narrativa de "ferramenta que supera o estado da arte" para "**sentimento financeiro em pt-BR precisa ser condicionado ao domínio; um classificador genérico é estruturalmente insuficiente; e um LLM barato + prompt de domínio atinge bom custo-benefício em FIIs e Ações**", assumindo abertamente a limitação em Renda Fixa.

---

## 6. ✅ Checklist de execução — ordenado por melhor relação esforço × impacto

> Ordem recomendada: **de cima para baixo**. O topo é "baixo esforço / alto impacto" (quick wins que sozinhos neutralizam a maior parte das críticas). Legenda: **Esf.** = esforço (Baixo/Médio/Alto) · **Imp.** = impacto · **Exp?** = exige experimento novo.

### 🟢 Bloco 1 — Quick wins (baixo esforço, alto/altíssimo impacto) — fazer primeiro

- [ ] **1. Corrigir a descrição de métricas e reportar macro-F1 + F1/precisão/recall por classe + weighted.** Os dados já existem (recomputados). Corrigir o texto que diz "macro" para refletir o que é reportado, e adicionar o macro (que hoje falta). *(Esf. Baixo · Imp. Altíssimo · Exp? Não)* — §5 métricas, Tab. 4. **[P0]**
- [ ] **2. Incluir baseline majoritário ("tudo-neutro") e matriz de confusão por domínio; reescrever a conclusão de Renda Fixa** (deixar de tratar 0,83 como sucesso; explicitar falha nas classes minoritárias). *(Esf. Baixo · Imp. Altíssimo · Exp? Não)* — §5, Conclusão. **[P0]**
- [ ] **3. Reenquadrar a comparação com FinBERT-PT-BR** como referência agnóstica de domínio / limite inferior; remover "supera/outperformed"; adicionar parágrafo de ameaça à validade. *(Esf. Baixo · Imp. Alto · Exp? Não)* — abstract, §5.2, Conclusão. **[P0]**
- [ ] **4. Informar o modelo/prompt/limite usados na geração dos resumos** (e declarar como ameaça se for um dos modelos avaliados). *(Esf. Baixo · Imp. Alto · Exp? Não)* — §3.1. **[P0]**
- [ ] **5. Reescrever a conclusão sobre o Sabiazinho** como trade-off de **custo** (~90% do macro-F1 do GPT-4o a ~7% do custo), não como "qualidade equivalente". *(Esf. Baixo · Imp. Alto · Exp? Não)* — §5, Conclusão. **[P1/independente]**
- [ ] **6. Trocar "ganhos >100%" / "acurácia >100%" por pontos percentuais** (ex.: 0,17→0,83 = +66pp). *(Esf. Baixo · Imp. Médio-Alto · Exp? Não)* — abstract, §5.2, Conclusão. **[P2]**
- [ ] **7. Corrigir as figuras 4 e 5 duplicadas** (uma deve ser `tempo_total_para_1884.pdf`). *(Esf. Baixo · Imp. Médio · Exp? Não)* — `main.tex` linhas ~664–681. **[apresentação/P0]**
- [ ] **8. Padronizar e completar referências** (`beltramini2024analise`, `darwish2025stockforecasting`, remover/citar `yan2023prediccao`). *(Esf. Baixo · Imp. Médio · Exp? Não)* — `sbc-template.bib`. **[P2]**
- [ ] **9. Ajustar fonte das tabelas** (Tab. 1 e demais com `\resizebox` excessivo). *(Esf. Baixo · Imp. Médio · Exp? Não)* **[P2]**
- [ ] **10. Trocar "transparência" por "fornecimento de justificativas textuais"** e remover linguagem promocional ("melhor modelo para a maioria das tarefas", "BECAUSE it is a smaller model"). *(Esf. Baixo · Imp. Médio · Exp? Não)* — Conclusão, §3.2, §4. **[P2]**
- [ ] **11. Bootstrap de IC 95% no macro-F1** por domínio/modelo (dados existem). *(Esf. Baixo · Imp. Médio-Alto · Exp? Não)* — §5. **[P1]**
- [ ] **12. Separar custo de inferência (tokens×preço) de latência de API**; remover o MacBook M2 como explicação de desempenho de modelos em nuvem. *(Esf. Baixo · Imp. Médio-Alto · Exp? Não)* — §5. **[P1]**

### 🟡 Bloco 2 — Médio esforço, alto impacto

- [ ] **13. Ablação de prompt em 1 modelo** (genérico vs domínio+role+one-shot completo; opcional condição intermediária) — **única evidência causal da tese central**. *(Esf. Médio · Imp. Alto · Exp? SIM, mínimo — ~35 min/condição)* — nova subseção em §5. **[P0]**
- [ ] **14. Adicionar o Sabiazinho 4** nas condições idênticas (mesmo dataset/prompts/temperatura 0) e reportar mesmo se pior — dá recomendação de produção viável no lugar do descontinuado. *(Esf. Baixo-Médio · Imp. Alto · Exp? SIM — 1 pipeline)* — §3.2/§5, Tabs. 3–4. **[P1]**
- [ ] **15. Custo-benefício objetivo:** tokens médios in/out, custo total do experimento, custo por notícia, projeção mensal (dados existem). *(Esf. Baixo-Médio · Imp. Médio-Alto · Exp? Não)* — §5. **[P1]**
- [ ] **16. Análise de erros de Ações:** matriz de confusão + leitura de ~20 exemplos errados; explicação objetiva (e corrigir a narrativa RF↔Ações via macro-F1). *(Esf. Médio · Imp. Médio · Exp? Não)* — §5. **[P1]**
- [ ] **17. Concordância entre anotadores detalhada:** % unanimidade / maioria de 3 / empate (dos 4 CSVs de anotação) + perfil/treinamento dos anotadores + guia de anotação disponibilizado/descrito. *(Esf. Baixo-Médio · Imp. Médio · Exp? Não)* — §3.1. **[P2]**

### 🟠 Bloco 3 — Médio esforço, impacto médio

- [ ] **18. Mini-estudo de fidelidade dos resumos:** 30–50 amostras, checagem manual + reanotação a partir do texto original, reportar taxa de mudança de rótulo. *(Esf. Médio · Imp. Médio-Alto · Exp? SIM, pequeno)* — §3.1. **[P1]**
- [ ] **19. Enunciar RQs formais** e mapear objetivo→metodologia→resultado→conclusão. *(Esf. Baixo · Imp. Médio · Exp? Não)* — Introdução/§3. **[P2]**
- [ ] **20. Trazer a essência dos prompts ao corpo/apêndice** e explicar o critério de escolha do exemplo one-shot. *(Esf. Baixo · Imp. Médio · Exp? Não)* — §3.2. **[P2]**
- [ ] **21. Discutir notícias de sinal misto** (regra de NEUTRO / efeitos que se anulam). *(Esf. Baixo · Imp. Médio · Exp? Não)* — §3.2. **[P2]**
- [ ] **22. Declarar limitações não discutidas:** janela temporal curta (24 dias), possível contaminação temporal dos modelos, indução determinística no prompt. *(Esf. Baixo · Imp. Médio · Exp? Não)* — Conclusão/Limitações. **[P2]**
- [ ] **23. Decidir o posicionamento da ferramenta:** reincluir a seção de tecnologias/UI (se AraraFin-ferramenta é contribuição) ou ajustar título/claims para "estudo comparativo". *(Esf. Baixo-Médio · Imp. Médio · Exp? Não)* — título, §4. **[P2]**

### ⚪ Bloco 4 — Opcional (P3) — NÃO fazer agora (escopo de novo trabalho)

- [ ] **24.** Avaliar fidelidade/alucinação de uma amostra das justificativas dos modelos. *(P3)*
- [ ] **25.** ~~Coletar corpora separados por ativo~~ — contradiz a tese; **não fazer**. *(responder aos revisores defendendo o desenho)*
- [ ] **26.** ~~Adaptar/re-treinar FinBERT multitarefa ou avaliação de sentimento geral com novo gabarito~~ — novo trabalho; **não fazer agora**.
- [ ] **27.** ~~Refazer o dataset com textos integrais~~ — escopo enorme; **não fazer**.

### 📌 Entregável transversal
- [ ] **28. Carta de resposta aos revisores** separando "corrigido" de "fora de escopo (com justificativa)". *(Esf. Médio · Imp. Alto para o resultado da submissão · Exp? Não)*

---

### Resumo de sequenciamento sugerido
1. **Sprint 1 (reanálise, sem experimento novo):** itens 1–12 + 15–17 + 19–23. Neutraliza ~70% das críticas e todos os riscos de recomputação.
2. **Sprint 2 (experimentos mínimos):** item 13 (ablação) + item 14 (Sabiazinho 4) + item 18 (fidelidade do resumo).
3. **Fechamento:** item 28 (carta de resposta) e revisão final de apresentação.
