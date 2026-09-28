# MVP 2 de SSD — Planejamento de Nova Unidade de Data Center

**Aluno:** Lucas Vilas Boas Iwano — 221021017
**Disciplina:** Sistemas de Suporte à Decisão — Universidade de Brasília (UnB)

Notebook de apoio à decisão para a abertura de uma nova unidade de uma empresa de armazenamento em nuvem, a partir de 100.000 registros de telemetria de data centers hyperscale (*Hyperscale Data Center Dataset*, Kaggle).

📓 Notebook: [`SSD_MVP2_Lucas_Iwano_221021017.ipynb`](SSD_MVP2_Lucas_Iwano_221021017.ipynb) (já executado, com gráficos e resultados salvos)

## O problema

Antes de ligar o primeiro servidor, a empresa escolhe **região**, **tipo de servidor**, **refrigeração** e **fonte de energia** (4 × 4 × 3 × 4 = 192 configurações). Cada uma é avaliada em quatro critérios:

| Critério | Variável |
|---|---|
| Risco de falha | `failure_risk_level` (probabilidade de risco *Alto*) |
| Custo | `compute_cost_usd` |
| Sustentabilidade | `carbon_emission_kg` |
| Tempo | `server_age_years` (janela de renovação) |

## Abordagem

1. **Três modelos** (risco, custo, carbono) que usam **só decisões de projeto** — telemetria de operação não existe antes da construção.
2. **Risco:** Regressão Logística vs. Gradient Boosting (`HistGradientBoostingClassifier`), validação cruzada estratificada, busca aleatória de hiperparâmetros, métrica F1 macro, importância por permutação.
3. **Simulação das 192 configurações**, fronteira de Pareto em 3 dimensões e score multicritério ponderado.
4. **Sensibilidade** sobre todo o simplex de pesos (231 combinações) e sobre o horizonte de planejamento.
5. **Cronograma de renovação** da configuração recomendada.

## Principais resultados

- **O risco é essencialmente função da idade:** 1–2% até o ano 7, 66% no ano 9, 100% nos anos 10–11.
- **A taxa bruta engana (confundimento):** APAC, refrigeração a ar e energia da rede parecem piores só porque têm frotas mais velhas. Controlando a idade, região, refrigeração e energia quase não mexem no risco; o tipo de servidor mexe (GPU chega ao ponto crítico um ano antes).
- **Custo e carbono dependem das decisões de projeto** — fonte de energia e região mudam muito a emissão.
- **Modelo de risco:** F1 macro 0,893 no teste (regressão logística 0,877; baseline 0,22), sem sinal de overfitting. Custo e carbono: R² de 0,93 e 0,92.
- **Recomendação global:** NA / Edge / Liquid / Hydro. Restrita ao perfil de armazenamento: NA / Storage / Liquid / Solar.
- **Cronograma:** atenção no ano 6; renovação no ano 9.

A recomendação é sensível à região e ao horizonte de planejamento; o notebook mostra essa sensibilidade em vez de escondê-la.

## Atenção: a região `NA`

A sigla `NA` (América do Norte) é lida como valor nulo pelo pandas por padrão, o que apagaria 28.801 registros. O notebook lê o CSV com `keep_default_na=False` e confere que a região foi preservada.

## Como executar

**Localmente**

```bash
pip install -r requirements.txt
jupyter notebook SSD_MVP2_Lucas_Iwano_221021017.ipynb
```

O arquivo `green_ai_datacenter.csv` deve estar na mesma pasta do notebook (já incluído neste repositório).

**No Google Colab:** abra o notebook, execute as células e faça o upload de `green_ai_datacenter.csv` quando solicitado.

## Limitações

Base sintética (o salto de risco entre os anos 8 e 10 é mais abrupto que o desgaste real); custo é computacional por registro, sem CAPEX; sem prazo de obra; regiões são agregados continentais; pesos do score são parâmetro de política. Detalhes na seção 10 do notebook.

## Estrutura

```
├── SSD_MVP2_Lucas_Iwano_221021017.ipynb   # análise completa
├── green_ai_datacenter.csv                # base de dados
├── requirements.txt
└── README.md
```

## Fonte dos dados

*Hyperscale Data Center Dataset* (Kaggle), arquivo `green_ai_datacenter.csv`. Os direitos pertencem ao autor original da base.
