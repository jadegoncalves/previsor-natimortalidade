# Previsor de natimortalidade municipal

Código e bases de análise do artigo **"Modelo preditor de natimortalidade municipal a partir de dados públicos do SUS: comparação com o IDHM e validação entre estados"**, de Jade de Vasconcelos Gonçalves e Sofia Mara Torres de Oliveira (UFMG, 2026).

O modelo indica quais municípios brasileiros tendem a ter taxa de natimortalidade acima da taxa do seu estado, usando apenas dados públicos do SUS. O resultado é comparado com duas referências: o IDHM e a taxa histórica do próprio município.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `modelo_preditor.ipynb` | Notebook com toda a análise: variáveis, seleção, treinamento, teste e comparações |
| `base_treino_2008_2015.csv` | Uma linha por município, com os dados de 2008 a 2015 somados |
| `base_teste_2016_2024.csv` | A mesma estrutura, com os dados de 2016 a 2024 |
| `previsoes_municipios.csv` | Risco previsto para cada município e a classe observada no período de teste (gerado pelo notebook) |

As bases já estão agregadas por município. O tratamento dos arquivos brutos do DATASUS não está neste repositório.

## Como executar

É necessário Python 3.11 ou mais recente.

```bash
pip install numpy pandas scipy scikit-learn statsmodels matplotlib seaborn notebook
jupyter notebook modelo_preditor.ipynb
```

Execute todas as células, na ordem. O notebook lê as duas bases da mesma pasta, cria a pasta `figuras/` com os gráficos em PDF e grava de novo o `previsoes_municipios.csv`. A execução leva alguns minutos, por causa das reamostragens usadas nos intervalos de confiança.

Os arquivos CSV usam ponto e vírgula como separador, vírgula como separador decimal e codificação UTF-8. Para abrir no pandas:

```python
import pandas as pd
base = pd.read_csv("base_treino_2008_2015.csv", sep=";", decimal=",", dtype={"CODMUNRES": str})
```

## Método, em resumo

- **Unidade de análise:** o município de residência da mãe. O Distrito Federal é analisado junto com Goiás, o que resulta em 26 grupos (coluna `GRUPO`).
- **Saída:** `ALTA` vale 1 quando a taxa de natimortalidade do município no período é maior que a taxa do seu estado, e 0 caso contrário.
- **Entradas:** dez indicadores candidatos, medidos em relação aos demais municípios do mesmo estado. A seleção manteve três: percentual de partos cesáreos, percentual de gestantes com sete ou mais consultas de pré-natal e percentual de mães com menos de oito anos de estudo.
- **Modelo:** regressão logística, comparada com Naive Bayes, KNN e árvore de decisão.
- **Teste:** os indicadores de 2008–2015 são usados para prever a classe de 2016–2024, sempre em um estado que ficou fora do treinamento. O procedimento é repetido para os 26 grupos.

As bases têm 5.570 municípios. A análise usa os 5.565 que têm IDHM de 2010.

## Principais resultados

Área sob a curva ROC (AUC) no período de teste, em média ponderada pelo número de municípios de cada estado:

| O que ordena os municípios | AUC |
|---|---|
| Só o IDHM | 0,587 |
| Só a taxa histórica do município | 0,582 |
| Modelo com indicadores do SUS | 0,623 |
| Modelo combinado com a taxa histórica | 0,631 |

O ganho do modelo sobre o IDHM foi de 0,036 (intervalo de confiança de 95%: 0,022 a 0,050) e ocorreu em 19 dos 26 grupos. O desempenho é modesto: o modelo serve para apoiar a priorização de municípios, e não para substituir a investigação local.

## Colunas das bases

| Coluna | Significado |
|---|---|
| `CODMUNRES` | Código do município no IBGE, com 6 dígitos |
| `MUNICIPIO`, `UF`, `REGIAO` | Nome, estado e região |
| `GRUPO` | Estado usado na análise (o DF aparece como GO) |
| `NASCIMENTOS` | Nascimentos totais no período (nascidos vivos mais óbitos fetais) |
| `OBITOS_FETAIS` | Óbitos fetais com 22 semanas ou mais de gestação, ou 500 g ou mais |
| `TAXA` | Óbitos fetais por mil nascimentos totais |
| `TAXA_UF` | A mesma taxa para o estado inteiro |
| `ALTA` | 1 se `TAXA` é maior que `TAXA_UF` |
| `EQUIPES_ESF_10MIL` | Equipes de Saúde da Família por 10 mil habitantes |
| `LOG_POPULACAO` | Logaritmo decimal da população |
| `PCT_PRENATAL_7MAIS` | % de gestantes com sete ou mais consultas de pré-natal |
| `PCT_PRENATAL_1TRIM` | % com pré-natal iniciado no primeiro trimestre (dados a partir de 2012) |
| `PCT_PRENATAL_ADEQUADO` | % com pré-natal adequado pelo índice de Kotelchuck (dados a partir de 2014) |
| `PCT_MAE_ADOLESCENTE` | % de mães adolescentes |
| `PCT_MAE_35MAIS` | % de mães com 35 anos ou mais |
| `PCT_MAE_ESC_MENOS8` | % de mães com menos de oito anos de estudo |
| `PCT_PRETA_PARDA` | % de nascidos pretos ou pardos |
| `PCT_CESAREA` | % de partos cesáreos |
| `IDHM`, `IDHM_E`, `IDHM_L`, `IDHM_R` | IDHM de 2010 e seus componentes: educação, longevidade e renda |

## Colunas das previsões

| Coluna | Significado |
|---|---|
| `TAXA_2008_2015`, `ALTA_2008_2015` | Taxa e classe do município no período de treinamento |
| `RISCO_PREVISTO` | Probabilidade de classe alta, estimada por um modelo que não usou o estado do município no treinamento |
| `POSICAO_NO_ESTADO` | Posição do risco previsto entre os municípios do estado, de 0 (menor) a 1 (maior) |
| `PREVISTO_ALTA` | Classe prevista, com o corte ajustado ao estado |
| `RISCO_SO_IDHM` | Probabilidade estimada pelo modelo que usa só o IDHM |
| `NASCIMENTOS_2016_2024`, `OBITOS_FETAIS_2016_2024`, `TAXA_2016_2024`, `ALTA_2016_2024` | Valores observados no período de teste |

## Fontes dos dados

- **SIM (DATASUS):** declarações de óbito fetal.
- **SINASC (DATASUS):** declarações de nascido vivo.
- **Relatórios de cobertura da atenção primária (Ministério da Saúde):** equipes de Saúde da Família e população.
- **Atlas do Desenvolvimento Humano no Brasil:** IDHM de 2010.

Todos os dados são públicos e estão agregados por município, sem identificação de pessoas.
