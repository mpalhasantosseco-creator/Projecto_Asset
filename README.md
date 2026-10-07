# Asset Pricing & Portfolio Management — Projeto de Grupo (2026/27)

NOVA IMS · Pós-Graduação em Data Science for Finance

Objetivo: estudar as propriedades empíricas dos retornos de ações (factos estilizados) e comparar
8 estratégias de construção de carteiras com um backtest *walk-forward* de 100 experiências.

## Estrutura do repositório

| Pasta / ficheiro | Conteúdo |
|---|---|
| `notebooks/01_coleta_e_tratamento.ipynb` | Universo (23 ações do S&P 500 + SPY + ^GSPC), download do Yahoo Finance, calendário, valores em falta, retornos log diários/semanais/mensais |
| `notebooks/02_analise_walmart.ipynb` | Análise exploratória da Walmart (WMT): preços, histogramas, Q-Q plots, quantis, VaR, ACF |
| `notebooks/03_modelacao_garch_volatilidade.ipynb` | Os 6 factos estilizados da WMT nas frequências diária, semanal e mensal (Ljung-Box, Jarque-Bera, skewness, ARCH-LM, GJR-GARCH, resíduos padronizados), com o SPY como comparação |
| `notebooks/04_estrategias_e_backtest.ipynb` | As 8 estratégias, ilustração com 23 ações, backtest de 100 experiências, métricas, robustez (custos, rebalanceamento trimestral, λ do Markowitz, períodos, ações) |
| `notebooks/data/` | Preços e retornos (CSV) gerados pelo notebook 01 e a taxa sem risco (FRED DGS3) |
| `graficos/` | Todos os gráficos (PNG). Os do notebook 03 começam por `03_`, os do 04 por `04_` |
| `resultados/` | Todas as tabelas em CSV e em Excel (`03_factos_estilizados.xlsx`, `04_estrategias_backtest.xlsx`) |
| `reports/` | Relatório |

## Como reproduzir

```bash
pip install -r requirements.txt
```

Correr os notebooks **por ordem**, a partir da pasta `notebooks/`:

1. `01` — descarrega os preços do Yahoo Finance (precisa de internet). Para não alterar os dados usados no relatório,
   a célula que grava os CSV está desativada; os CSV em `notebooks/data/` são os da extração original.
2. `02` e `03` — só leem os CSV.
3. `04` — na primeira execução descarrega a série **DGS3** do FRED e grava `data/taxa_sem_risco_DGS3.csv`
   (nas seguintes lê o ficheiro local). Demora cerca de 1–2 minutos.

## Dados

- **Preços:** Yahoo Finance via `yfinance`, preço de fecho ajustado (*Adj Close*, inclui dividendos e *splits*),
  de 2010-01-04 a 2026-09-18 (extração em setembro de 2026; `fim = 2026-09-21` no notebook 01). Calendário = dias de negociação do SPY.
  A TSLA só tem dados a partir de 29/06/2010 (122 dias em falta no início).
- **Taxa sem risco:** *yield* da Treasury americana a 3 anos, FRED série `DGS3` (% ao ano).
- **SPY vs ^GSPC:** o SPY é o ETF investível (benchmark de mercado no backtest); o ^GSPC é o índice, não investível e sem dividendos.

## Convenções do backtest (notebook 04)

| Item | Escolha |
|---|---|
| Experiências | 100, semente `2026` |
| Cada experiência | período contíguo de 3 anos (756 dias úteis) + 12 ações sorteadas entre as 23 elegíveis |
| Estimação / avaliação | anos 1–2 / ano 3 (fora da amostra) |
| Janela de estimação | móvel de 1 ano (252 dias), só dados passados |
| Rebalanceamento | semestral (trimestral como teste de robustez); pesos derivam entre datas |
| Restrições | long-only, totalmente investido, sem alavancagem |
| Custos | 10 bps sobre Σ\|Δw\| (robustez: 0, 25, 50 bps) |
| Benchmarks | Equally weighted e SPY buy & hold |
| Markowitz | aversão ao risco λ = 3 (sensibilidade com λ = 1 e 5) |

## Correspondência com o relatório

| Secção do relatório | Onde está |
|---|---|
| 1. Universo e construção da carteira | notebook 01 |
| 2.1.1 Autocorrelação | notebook 03, secção 3 — `03_acf_retornos_wmt.png`, `03_ljung_box_retornos.csv` |
| 2.1.2 Não-normalidade / caudas pesadas | notebook 03, secção 4 — `03_qqplot_normal_wmt_3freq.png`, `03_normalidade.csv` |
| 2.1.3 Assimetria | notebook 03, secção 5 — `03_assimetria.csv` |
| 2.1.4 Clustering de volatilidade | notebook 03, secção 6 — `03_acf_abs_quadrado_wmt.png`, `03_volatilidade_movel_wmt.png`, `03_clustering.csv` |
| 2.1.5 Efeito alavanca | notebook 03, secção 7 — `03_news_impact_curve.png`, `03_correlacao_alavanca.png`, `03_gjr_garch.csv` |
| 2.1.6 Não-normalidade condicional | notebook 03, secção 8 — `03_qqplot_residuos_padronizados_wmt.png`, `03_residuos_padronizados.csv` |
| Resumo dos factos estilizados | notebook 03, secção 9 — `03_resumo_wmt.csv` |
| 3. Estratégias (objetivos e restrições) | notebook 04, secções 3–4 — `04_pesos_ilustracao.png`, `04_fronteira_eficiente.png`, `04_contribuicoes_risco_iv_vs_erc.png` |
| 3. Resultados do backtest | notebook 04, secção 9 — `04_mediana_metricas.csv`, `04_bate_ew.csv`, `04_boxplots_metricas.png` |
| Robustez e limitações | notebook 04, secções 10 e 12 |
