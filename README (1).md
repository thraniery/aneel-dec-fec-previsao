# Previsão de Risco de Violação dos Indicadores de Continuidade (DEC/FEC)

Projeto Integrador em Ciência de Dados — UniCEUB
Thiago Raniery e Gabriel Braga

## Contexto

A ANEEL fiscaliza a qualidade do fornecimento de energia elétrica no Brasil por meio de dois indicadores, apurados mensalmente por conjunto de unidades consumidoras:

- **DEC** (Duração Equivalente de Interrupção por Unidade Consumidora): número médio de horas que cada consumidor ficou sem energia no período.
- **FEC** (Frequência Equivalente de Interrupção por Unidade Consumidora): número médio de vezes que cada consumidor teve o fornecimento interrompido no período.

Cada conjunto de unidades consumidoras possui limites regulatórios definidos pela ANEEL para DEC e FEC. Quando uma distribuidora ultrapassa esses limites, ela é obrigada a compensar financeiramente os consumidores afetados. Hoje, esse acompanhamento é majoritariamente reativo: a ação corretiva ocorre depois que a violação já aconteceu.

## Problema e objetivo

**Pergunta de pesquisa:** é possível prever quais conjuntos de unidades consumidoras têm maior risco de ultrapassar os limites regulatórios de DEC/FEC, a partir do seu histórico de indicadores?

**Objetivo geral:** desenvolver um modelo de classificação capaz de prever o risco de violação dos indicadores de continuidade por conjunto de unidades consumidoras, permitindo ação preventiva em vez de reativa.

**Por que importa:** antecipar violações reduz o custo com compensações financeiras e com manutenção corretiva emergencial, além de melhorar a qualidade do serviço prestado ao consumidor.

## Escopo e dados

| Item | Descrição |
|---|---|
| Fonte | ANEEL — Portal de Dados Abertos, dataset "Indicadores Coletivos de Continuidade (DEC e FEC)" |
| Recorte geográfico | Região Centro-Oeste (DF, GO, MT, MS) |
| Distribuidoras | Neoenergia Brasília (DF), Equatorial Goiás (GO), EMT — Energisa Mato Grosso (MT), EMS — Energisa Mato Grosso do Sul (MS) |
| Recorte temporal | Janeiro/2020 a julho/2026 |
| Granularidade | Mensal, por conjunto de unidades consumidoras |
| Volume | 26.062 linhas · 377 conjuntos |

### Dicionário de dados (principais colunas)

| Coluna | Descrição |
|---|---|
| `UF` | Unidade federativa da distribuidora |
| `SigAgente` | Sigla da distribuidora |
| `id_conjunto` | Identificador do conjunto de unidades consumidoras |
| `conjunto` | Nome do conjunto de unidades consumidoras |
| `data` | Mês de referência do indicador |
| `DEC` | Duração Equivalente de Interrupção apurada no mês (horas) |
| `FEC` | Frequência Equivalente de Interrupção apurada no mês (nº de interrupções) |
| `num_uc` | Número de unidades consumidoras do conjunto |
| `DECINC`, `DECIND`, `DECINE`, `DECINO`, `DECIP`, `DECIPC`, `DECXN`, `DECXNC`, `DECXP`, `DECXPC` | Parcelas do DEC desagregadas por origem da interrupção (interna, externa, programada, não programada etc.) |
| `FECINC`, `FECIND`, `FECINE`, `FECINO`, `FECIP`, `FECIPC`, `FECXN`, `FECXNC`, `FECXP`, `FECXPC` | Parcelas equivalentes do FEC |

As colunas `DECXNC`, `DECXPC`, `FECXNC` e `FECXPC` só possuem valores em 2020-2021; a metodologia da ANEEL descontinuou esses indicadores a partir de 2022.

Metodologicamente, só contam para o DEC/FEC as interrupções com duração superior a 3 minutos (ANEEL, PRODIST — Módulo 8), admitidos alguns expurgos na apuração.

## Estrutura do repositório

```
├── data/
│   └── raw/          # bases originais extraídas da ANEEL (xlsx e csv)
├── notebooks/         # análise exploratória e experimentos
├── src/                # scripts de tratamento e modelagem
├── slides/             # material de apresentação
└── README.md
```

## Metodologia

1. **Base histórica** — consolidação de DEC/FEC por conjunto e mês a partir dos dados abertos da ANEEL. *(concluído)*
2. **Variável-alvo** — cruzamento com os limites regulatórios por conjunto, gerando a marcação de violação (sim/não).
3. **Enriquecimento** — incorporação de atributos físico-elétricos dos conjuntos e, se aplicável, variáveis climáticas.
4. **Análise exploratória** — padrões por conjunto, distribuidora, sazonalidade e período.
5. **Modelagem** — treinamento e comparação de classificadores (Regressão Logística, Random Forest/XGBoost).
6. **Avaliação** — métricas de desempenho (precisão, recall, AUC-ROC) e interpretação dos fatores mais relevantes.
7. **Dashboard** — visualização interativa dos resultados.

## Status atual

- [x] Tema e pergunta de pesquisa definidos
- [x] Fonte de dados oficial identificada
- [x] Base bronze consolidada (Centro-Oeste, 2020-2026)
- [ ] Variável-alvo (cruzamento com limites regulatórios)
- [ ] Enriquecimento com atributos dos conjuntos
- [ ] Modelagem e avaliação
- [ ] Dashboard final

## Tecnologias

Python (pandas, scikit-learn), Power BI, Excel.

## Fonte dos dados

ANEEL — Portal de Dados Abertos: https://dadosabertos.aneel.gov.br
