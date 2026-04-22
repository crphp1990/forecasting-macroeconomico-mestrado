# Comparação de Modelos de Forecasting para Séries Macroeconômicas Mensais

Repositório de códigos, dados e resultados da dissertação de mestrado que
compara o desempenho preditivo de 14 modelos de forecasting aplicados a 29
séries macroeconômicas brasileiras e internacionais, observadas entre
janeiro de 2017 e dezembro de 2025.

**Autor:** Carlos Phillip Ruiz
**Instituição:** EESP-FGV — Mestrado em Finanças
**Contato:** c376018@fgv.edu.br

## Sobre o trabalho

A dissertação avalia, sob um mesmo protocolo empírico, o desempenho de
cinco grupos metodológicos:

- **Estatísticos e econométricos:** ARIMA, SARIMAX, ETS, Prophet, Theta
- **Volatilidade condicional:** ARCH, GARCH
- **Aprendizado de máquina:** XGBoost, RandomForest
- **Modelos pré-treinados:** Chronos, Lag-Llama, TimesFM
- **Benchmarks naive:** Random Walk, Seasonal Naive

O protocolo empírico utiliza walk-forward expansível com 24 meses de teste
e horizonte de previsão de três meses. A comparação é feita por MAPE
(métrica principal), MAE e RMSE, com testes estatísticos de Friedman,
pós-teste de Nemenyi e Diebold-Mariano com correção de Newey-West.

## Estrutura do repositório

```
forecasting-macroeconomico-mestrado/
├── README.md
├── requirements.txt
├── LICENSE
├── data/
│   └── base_economica_brasil.csv
├── notebooks/
│   ├── Forecast_busca_bases_macroeconomia.ipynb
│   ├── Forecasting_analise_exploratoria.ipynb
│   ├── Forecasting_ARIMA.ipynb
│   ├── Forecasting_SARIMAX.ipynb
│   ├── Forecasting_ETS.ipynb
│   ├── Forecasting_Theta.ipynb
│   ├── Forecasting_Prophet.ipynb
│   ├── Forecasting_ARCH.ipynb
│   ├── Forecasting_GARCH.ipynb
│   ├── Forecasting_XGBoost.ipynb
│   ├── Forecasting_RandomForest.ipynb
│   ├── Forecasting_Chronos.ipynb
│   ├── Forecasting_LagLlama.ipynb
│   ├── Forecasting_TimesFM.ipynb
│   ├── Forecasting_TEMPO.ipynb
│   └── Forecast_comparacao_modelo.ipynb
└── resultados/
    ├── resultados_<modelo>.csv
    ├── previsoes_<modelo>.csv
    └── consolidado_*.png
```

## Requisitos gerais

- Python 3.10 ou superior
- Bibliotecas listadas em `requirements.txt`
- Aproximadamente 2 GB de espaço em disco para os modelos pré-treinados

Instalação das bibliotecas base:

```bash
pip install -r requirements.txt
```

## Reprodutibilidade

Todos os modelos com componentes estocásticos utilizam semente fixada
(`SEED = 42`), aplicada a `random`, `numpy` e `torch`. O pipeline completo
foi validado em execução integral (16 notebooks) sem erros.

Modelos determinísticos (ARIMA, SARIMAX, ETS, Theta) não exigem semente,
mas foram executados no mesmo ambiente para garantir consistência.

## Configuração dos modelos pré-treinados

Os modelos pré-treinados (Chronos, Lag-Llama, TimesFM e TEMPO) têm
requisitos de instalação específicos que exigem atenção. Esta seção
descreve o procedimento adotado para cada um.

### Chronos (Amazon)

Instalação direta via pip:

```bash
pip install chronos-forecasting
```

O modelo `amazon/chronos-t5-small` (46M parâmetros) é baixado
automaticamente na primeira execução a partir do HuggingFace Hub.

### TimesFM (Google)

Instalação via pip:

```bash
pip install timesfm
```

Checkpoint utilizado: `google/timesfm-1.0-200m-pytorch` (200M parâmetros),
também baixado automaticamente na primeira execução.

### Lag-Llama (ServiceNow Research)

O Lag-Llama exige configuração manual, pois não está disponível como pacote
pip padrão. O procedimento utilizado foi:

1. Clonar o repositório oficial:

```bash
git clone https://github.com/time-series-foundation-models/lag-llama.git
```

2. Baixar o checkpoint pré-treinado do HuggingFace Hub:

```bash
huggingface-cli download time-series-foundation-models/Lag-Llama lag-llama.ckpt --local-dir lag-llama
```

3. Instalar as dependências específicas do Lag-Llama (as versões importam):

```bash
cd lag-llama
pip install -r requirements.txt
cd ..
```

4. O notebook `Forecasting_LagLlama.ipynb` utiliza `sys.path.append('..')`
   para importar o módulo local do repositório clonado. A estrutura
   esperada é:

```
seu-projeto/
├── lag-llama/
│   ├── lag-llama.ckpt
│   ├── lag_llama/
│   └── requirements.txt
└── notebooks/
    └── Forecasting_LagLlama.ipynb
```

**Observações importantes sobre o Lag-Llama:**

- As dependências do Lag-Llama incluem versões específicas de PyTorch e
  GluonTS listadas no `requirements.txt` do repositório oficial. Versões
  mais recentes do PyTorch podem ser incompatíveis com o checkpoint.
- A API utilizada é a `LagLlamaEstimator` do GluonTS, acessada via
  importação local.
- O uso de um ambiente virtual dedicado ao Lag-Llama é fortemente
  recomendado para evitar conflitos com outras bibliotecas do projeto
  (em especial com o PyTorch utilizado pelo Chronos e TimesFM).

### TEMPO (excluído da análise final)

O modelo TEMPO foi excluído da análise comparativa final da dissertação por
limitações técnicas identificadas durante a implementação: dupla
normalização RevIN e incompatibilidade entre o comprimento de sequência
esperado (336 pontos) e a extensão das séries mensais disponíveis (108
observações). O notebook está mantido no repositório para fins de
transparência metodológica.

A reprodução do notebook exige ambiente virtual isolado (`venv_tempo`) com
o repositório oficial clonado e checkpoint próprio:

```bash
python -m venv venv_tempo
source venv_tempo/bin/activate  # Linux/Mac
venv_tempo\Scripts\activate     # Windows

git clone https://github.com/DC-research/TEMPO.git
pip install torch==1.13.0
pip install -r TEMPO/requirements.txt
```

Checkpoint utilizado: `Melady/TEMPO/TEMPO-80M_v1.pth` (80M parâmetros).

## Como reproduzir o pipeline completo

Os notebooks devem ser executados na seguinte ordem:

1. `Forecast_busca_bases_macroeconomia.ipynb` — coleta e consolidação da
   base de dados a partir das fontes originais (BCB, IBGE, FGV, FRED,
   Yahoo Finance).
2. `Forecasting_analise_exploratoria.ipynb` — análise descritiva das
   séries (estatísticas, testes de estacionariedade, sazonalidade).
3. Notebooks de modelo (ordem indiferente entre si):
   - `Forecasting_ARIMA.ipynb`
   - `Forecasting_SARIMAX.ipynb`
   - `Forecasting_ETS.ipynb`
   - `Forecasting_Theta.ipynb`
   - `Forecasting_Prophet.ipynb`
   - `Forecasting_ARCH.ipynb`
   - `Forecasting_GARCH.ipynb`
   - `Forecasting_XGBoost.ipynb`
   - `Forecasting_RandomForest.ipynb`
   - `Forecasting_Chronos.ipynb`
   - `Forecasting_LagLlama.ipynb` (requer setup específico — ver acima)
   - `Forecasting_TimesFM.ipynb`
   - `Forecasting_Naive.ipynb`
4. `Forecast_comparacao_modelo.ipynb` — consolidação dos resultados,
   testes estatísticos e geração das figuras.

Cada notebook de modelo é autônomo, lê a base `base_economica_brasil.csv` e
gera dois arquivos: `resultados_<modelo>.csv` e `previsoes_<modelo>.csv`.
O notebook de comparação consolida todos esses arquivos.

## Tempos de execução aproximados

Referência do pipeline completo executado em 2026-04-21:

| Notebook | Tempo |
|---|---|
| ARIMA | 22 min |
| SARIMAX | 2 min |
| ETS | 40 s |
| Theta | 15 s |
| Prophet | 3 min |
| ARCH | 4 min |
| GARCH | 10 min |
| XGBoost | 5 min |
| RandomForest | 6 min |
| Chronos | 2 min |
| Lag-Llama | 2 min |
| TimesFM | 9 min |
| TEMPO | 1 min |
| Consolidação | 40 s |

Total aproximado: 70 minutos em máquina com CPU padrão (sem GPU). Os
modelos pré-treinados rodam em CPU por padrão; uso de GPU reduz tempos
de inferência.

## Dados

A base consolidada `base_economica_brasil.csv` contém 29 séries
macroeconômicas mensais observadas entre janeiro de 2017 e dezembro de
2025 (108 observações por série).

**Fontes originais:**

- Banco Central do Brasil (BCB/SGS): IBC-Br, Selic, Câmbio, M2, Crédito,
  Inadimplência, Dívida/PIB
- IBGE: PMS Volume, Vendas Varejo, Massa Salarial, Desemprego, INCC
- FGV: NUCI, ICE Empresarial, ICC
- Federal Reserve (FRED): Prod. Industrial EUA, CPI EUA, Housing Starts,
  Dollar Index
- Yahoo Finance: S&P 500, Ibovespa, ETF Emergentes, Brent, Ouro, Cobre,
  Café, Soja, Gás Natural, Minério

O procedimento de coleta e harmonização está documentado no notebook
`Forecast_busca_bases_macroeconomia.ipynb`.

## Limitações conhecidas

- **Componentes estocásticos:** apesar do uso de seed fixa, algumas
  bibliotecas (Optuna, Prophet) podem apresentar variações numéricas
  mínimas entre execuções em ambientes com configurações de hardware
  diferentes. Essas variações não alteram o padrão qualitativo das
  conclusões.
- **Versões de bibliotecas:** os resultados reportados referem-se às
  versões listadas em `requirements.txt`. Atualizações de `statsmodels`,
  `pmdarima`, `arch` e `torch` podem afetar marginalmente os valores
  exatos.
- **Séries específicas:** modelos de volatilidade (ARCH, GARCH) foram
  aplicados apenas a séries com pelo menos 84 observações. Modelos
  estatísticos com sazonalidade utilizam `m=12` fixo.

## Contato

**Carlos Phillip Ruiz**
EESP-FGV — Mestrado em Finanças
c376018@fgv.edu.br

Dúvidas sobre o código ou sugestões de melhoria podem ser encaminhadas
por e-mail ou via sistema de issues do GitHub.

## Citação

Caso utilize este código ou os resultados em trabalhos acadêmicos, favor
citar:

> RUIZ, Carlos Phillip. Comparação de modelos de forecasting para séries
> macroeconômicas mensais. 2026. Dissertação (Mestrado em Finanças) —
> Escola de Economia de São Paulo, Fundação Getulio Vargas, São Paulo, 2026.

## Licença

Este trabalho está disponibilizado sob a licença MIT. Consulte o arquivo
`LICENSE` para mais detalhes.
