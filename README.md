# Previsão de Risco de Violação dos Indicadores de Continuidade (DEC/FEC)

Projeto Integrador em Ciência de Dados — UniCEUB
Autores: Thiago Raniery e Gabriel Braga

Situação: em desenvolvimento (outubro de 2026)

---

## Sumário

1. [Visão geral](#visão-geral)
2. [Problema e pergunta de pesquisa](#problema-e-pergunta-de-pesquisa)
3. [Dados](#dados)
4. [Primeiras análises](#primeiras-análises)
5. [Metodologia](#metodologia)
6. [Status do projeto](#status-do-projeto)
7. [Estrutura do repositório](#estrutura-do-repositório)
8. [Como reproduzir o tratamento](#como-reproduzir-o-tratamento)
9. [Tecnologias](#tecnologias)
10. [Fonte e referências](#fonte-e-referências)

---

## Visão geral

A ANEEL fiscaliza a qualidade do fornecimento de energia elétrica no Brasil por meio de dois indicadores, apurados mensalmente para cada **conjunto de unidades consumidoras** (a área de atendimento de uma distribuidora, que agrupa consumidores de uma mesma região):

- **DEC** (Duração Equivalente de Interrupção por Unidade Consumidora): número médio de horas que cada consumidor ficou sem energia no período.
- **FEC** (Frequência Equivalente de Interrupção por Unidade Consumidora): número médio de vezes que cada consumidor teve o fornecimento interrompido no período.

Cada conjunto possui limites regulatórios de DEC e FEC definidos pela ANEEL. Quando a distribuidora ultrapassa esses limites, ela é obrigada a compensar financeiramente os consumidores afetados. Hoje, esse acompanhamento é majoritariamente reativo: a ação corretiva acontece depois que a violação já ocorreu.

Metodologicamente, só contam para o DEC/FEC as interrupções com duração superior a 3 minutos (ANEEL, PRODIST — Módulo 8), admitidos alguns expurgos na apuração.

## Problema e pergunta de pesquisa

**Pergunta de pesquisa**

> Quais áreas de atendimento (conjuntos) do Brasil têm maior risco de violar os limites de DEC/FEC, com base no histórico de 2010–2026?

**Objetivo geral**

Desenvolver um modelo de classificação capaz de estimar o risco de violação dos limites de DEC/FEC por conjunto, gerando um ranking de áreas por risco que permita ação preventiva em vez de reativa.

**Objetivos específicos**

1. Construir uma base histórica nacional, com no mínimo 15 anos, de DEC/FEC por conjunto e mês.
2. Cruzar a base com os limites regulatórios para criar a variável-alvo (violou ou não violou).
3. Explorar os dados: distribuição, tendência, sazonalidade e concentração das violações.
4. Treinar e comparar modelos de classificação, avaliando-os com validação temporal.
5. Identificar as variáveis mais relevantes para o risco e apresentar o resultado em um painel.

**Por que importa**

Antecipar violações pode reduzir o custo com compensações financeiras e com manutenção corretiva emergencial, além de melhorar a qualidade do serviço prestado ao consumidor.

## Dados

**Fonte:** ANEEL — Portal de Dados Abertos, conjunto de dados "Indicadores Coletivos de Continuidade (DEC e FEC)", atualizado mensalmente e com cobertura a partir de 2010.

**Escopo**

| Item | Meta do projeto | Já tratado |
|---|---|---|
| Recorte geográfico | Brasil | Região Centro-Oeste (DF, GO, MT, MS) |
| Período | 2010 a 2026 (mais de 15 anos) | Janeiro/2020 a julho/2026 |
| Granularidade | Mensal, por conjunto | Mensal, por conjunto |
| Volume | A definir após incorporar 2010-2019 | 26.062 linhas, 377 conjuntos |

O arquivo nacional de 2020 a 2026 tem cerca de 5,1 milhões de registros e 105 distribuidoras (formato longo, com vários indicadores por conjunto e mês). O arquivo de 2010 a 2019 ainda será incorporado.

**Distribuidoras já tratadas**

| UF | Distribuidora | Conjuntos | Linhas |
|---|---|---|---|
| DF | Neoenergia Brasília | 27 | 2.006 |
| GO | Equatorial Goiás | 175 | 12.045 |
| MT | EMT — Energisa Mato Grosso | 108 | 7.240 |
| MS | EMS — Energisa Mato Grosso do Sul | 67 | 4.771 |

### Dicionário de dados

Base tratada: `data/raw/base_bronze_dec_fec_centrooeste.xlsx` e `.csv` (uma linha por conjunto e mês).

| Coluna | Descrição |
|---|---|
| `UF` | Unidade federativa da distribuidora |
| `SigAgente` | Sigla da distribuidora |
| `id_conjunto` | Identificador do conjunto de unidades consumidoras |
| `conjunto` | Nome do conjunto |
| `data` | Mês de referência |
| `DEC` | DEC apurado no mês (horas) |
| `FEC` | FEC apurado no mês (número de interrupções) |
| `num_uc` | Número de unidades consumidoras do conjunto |
| `DECINC`, `DECIND`, `DECINE`, `DECINO`, `DECIP`, `DECIPC`, `DECXN`, `DECXNC`, `DECXP`, `DECXPC` | Parcelas do DEC por origem da interrupção |
| `FECINC`, `FECIND`, `FECINE`, `FECINO`, `FECIP`, `FECIPC`, `FECXN`, `FECXNC`, `FECXP`, `FECXPC` | Parcelas equivalentes do FEC |

O significado de cada sigla de parcela consta no arquivo de domínio dos indicadores, publicado pela ANEEL junto ao conjunto de dados.

### Limitações conhecidas

- Os dados vêm em formato longo (conjunto, indicador e mês em linhas) e precisam ser reestruturados antes da análise.
- Códigos de conjuntos e nomes de distribuidoras mudam ao longo dos anos. Exemplo: o conjunto Mangueiral aparece com um código até dezembro de 2021 e com outro a partir de janeiro de 2022.
- As colunas `DECXNC`, `DECXPC`, `FECXNC` e `FECXPC` só têm valores em 2020 e 2021; a ANEEL descontinuou esses indicadores depois disso.
- 2026 é parcial (dados até julho), o que afeta comparações de médias anuais.
- As parcelas de DEC/FEC compõem o próprio indicador. Por isso, as variáveis explicativas do modelo serão construídas apenas com informação anterior ao período previsto, para evitar vazamento de dados.
- O horizonte da previsão (mês ou ano) depende do formato do arquivo de limites regulatórios e será definido ao cruzá-lo com a base.

## Primeiras análises

Resultados exploratórios sobre a base do Centro-Oeste (2020 a julho/2026). São descrições dos dados, não resultados do modelo.

**Distribuição do DEC mensal por conjunto**

| Medida | Valor |
|---|---|
| Média | 1,86 h |
| Mediana | 1,06 h |
| Máximo | 53,6 h |
| Registros com DEC abaixo de 1 h | 47,7% |
| Registros com DEC acima de 5 h | 7,6% |

A média bem acima da mediana indica uma cauda de casos extremos: a maior parte dos meses tem bom serviço, mas poucos registros muito ruins puxam a média para cima. São esses casos que o modelo tentará antecipar.

**Média por distribuidora**

| UF | DEC médio (h) | FEC médio |
|---|---|---|
| DF | 0,75 | 0,49 |
| GO | 2,18 | 0,86 |
| MT | 2,03 | 0,79 |
| MS | 1,27 | 0,48 |

**Evolução anual (média simples entre conjuntos)**

| Ano | DEC médio (h) | FEC médio |
|---|---|---|
| 2020 | 1,91 | 0,81 |
| 2021 | 2,19 | 0,84 |
| 2022 | 2,25 | 0,85 |
| 2023 | 2,19 | 0,87 |
| 2024 | 1,59 | 0,65 |
| 2025 | 1,41 | 0,59 |
| 2026 (parcial) | 1,12 | 0,47 |

Os indicadores caem a partir de 2023, mas 2026 só inclui dados até julho e não deve ser comparado diretamente com anos completos.

## Metodologia

1. **Base histórica (bronze):** consolidação de DEC/FEC por conjunto e mês a partir dos dados abertos da ANEEL.
2. **Variável-alvo:** cruzamento com os limites regulatórios por conjunto, gerando a marcação de violação (sim ou não).
3. **Preparação (silver/gold):** padronização de códigos e nomes entre períodos, tratamento de valores ausentes e criação de variáveis explicativas (histórico recente do conjunto, distribuidora, estado, porte e época do ano).
4. **Análise exploratória:** padrões por conjunto, distribuidora, sazonalidade e período.
5. **Modelagem:** Regressão Logística como referência e Random Forest ou XGBoost como modelos mais flexíveis, com separação temporal entre treino e teste (treina no passado, testa no futuro).
6. **Avaliação:** precisão, recall, F1 e AUC-ROC, considerando o desbalanceamento esperado entre conjuntos que violam e que não violam, e interpretação das variáveis mais relevantes.
7. **Painel:** ranking e visualização do risco por conjunto.

Clima e características físico-elétricas da rede são extensões possíveis, condicionadas à disponibilidade e à cobertura temporal desses dados.

## Status do projeto

Pronto

- [x] Tema, pergunta de pesquisa e escopo definidos
- [x] Fonte oficial mapeada
- [x] Base tratada do Centro-Oeste (2020-2026)
- [x] Análise exploratória inicial

Em andamento

- [ ] Ampliação da base para o Brasil, com mais de 15 anos
- [ ] Cruzamento com os limites regulatórios (variável-alvo)

Planejado

- [ ] Enriquecimento com variáveis explicativas
- [ ] Modelagem e avaliação
- [ ] Painel de risco por conjunto
- [ ] Validação dos resultados com o parceiro de extensão

## Estrutura do repositório

```
aneel-dec-fec-previsao/
├── data/
│   └── raw/          # bases tratadas a partir dos dados da ANEEL (xlsx e csv)
├── notebooks/        # análise exploratória e experimentos
├── src/              # scripts de tratamento e modelagem
├── slides/           # material de apresentação
└── README.md
```

## Como reproduzir o tratamento

O tratamento segue sempre os mesmos passos:

1. Baixar os arquivos de indicadores no [conjunto de dados da ANEEL](https://dadosabertos.aneel.gov.br/dataset/indicadores-coletivos-de-continuidade-dec-e-fec) (2010-2019 e 2020-2029) e o arquivo de limites regulatórios.
2. Descompactar os arquivos ZIP e ler os CSV (separador ponto e vírgula).
3. Converter os valores numéricos (vírgula decimal) e reestruturar o formato longo em uma linha por conjunto e mês, com uma coluna por indicador.
4. Cruzar com os limites regulatórios para gerar a variável-alvo.

A base atual do Centro-Oeste foi gerada com os passos 1 a 3. Os scripts de tratamento da base nacional serão publicados na pasta `src/` quando essa etapa for concluída.

<!--
Seção de extensão (ACE) — preencher quando o parceiro estiver definido e remover este comentário:

## Parceiro de extensão
- Organização:
- Responsável:
- Contatos realizados:
- Termo de Anuência:
-->

## Tecnologias

Python (pandas, scikit-learn), Power BI e Excel.

## Fonte e referências

- ANEEL — Portal de Dados Abertos: [Indicadores Coletivos de Continuidade (DEC e FEC)](https://dadosabertos.aneel.gov.br/dataset/indicadores-coletivos-de-continuidade-dec-e-fec)
- ANEEL — PRODIST, Módulo 8 (Qualidade do Fornecimento de Energia Elétrica)
