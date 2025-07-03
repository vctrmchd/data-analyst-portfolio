# Análise Exploratória de Dados das Colheitas do Sítio Mãeã Natureza

## Descrição do Projeto

Este projeto consiste em uma análise exploratória de dados de registros de colheita do Sítio Mãeã Natureza. O objetivo principal é extrair insights valiosos sobre os padrões de produção ao longo do tempo, identificar sazonalidades e a diversidade de cultivos, e assim, propor melhorias e aprofundamentos para a gestão agrícola.

## Principais Descobertas

* Sazonalidade da Produção: A análise revelou uma clara variação na produção de diferentes cultivos ao longo das estações do ano, destacando a natureza cíclica da agricultura no Sítio Mãeã Natureza. Este insight é crucial para otimizar o planejamento de plantio e colheita.
* Importância da Qualidade dos Dados: Uma fase extensiva de limpeza e padronização dos dados, especialmente a separação e categorização de "cultivo" e "variedade", foi fundamental para garantir a consistência, a precisão e a confiabilidade das análises seguintes.
* Diversidade e Volume de Cultivos: O sítio mantém uma produção diversificada, com a identificação dos cultivos mais frequentes e/ou com maior volume de colheita, que variam significativamente em sua presença e abundância ao longo do tempo.
* Relação Clima-Produção: Observou-se que a relação entre variáveis climáticas (como temperatura e precipitação) e a produção varia significativamente entre os diferentes cultivos, indicando que a resposta à chuva, por exemplo, pode ser inversa para certos itens como manga e banana em comparação com mandioca e tomate.
* Estabilidade da Produção Anual: Apesar das flutuações sazonais de cultivos específicos, o volume total de produção ao longo das estações (Verão, Outono, Inverno, Primavera) mostrou uma variação relativamente baixa, sugerindo que a diversidade de cultivos ajuda a manter uma oferta constante.
* Comportamento dos Coagricultores: A maioria dos coagricultores tende a permanecer por períodos curtos (1 a 6 meses), contribuindo com faturamentos totais relativamente baixos, o que indica um desafio na fidelização e engajamento a longo prazo.

## Tecnologias Utilizadas

* **Linguagem:** R
* **Bibliotecas (R):**
    * `dplyr`: Para manipulação e transformação de dados.
    * `readr`: Para importação de dados CSV.
    * `lubridate`: Para manipulação de datas.
    * `stringi`, `stringr`: Para manipulação e limpeza de strings de texto.
    * `tidyr`: Para reorganização de dados (ex: `separate`, `unnest`).
    * `janitor`: Para limpeza e padronização de nomes de colunas.
    * `ggplot2`: Para criação de gráficos estatísticos.
    * `scales`: Para formatação de escalas em gráficos.
    * `knitr`: Para tabelas.
* **Ferramentas:** RStudio / R Markdown
