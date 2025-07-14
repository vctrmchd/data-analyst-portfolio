Documentação para Dashboard – Sítio Mãeã Natureza
================
Victor Machado


# Introdução  
  
Este documento serve como um guia detalhado para a recriação e manutenção do dashboard "Mãeã Natureza: Desempenho da Produção" no Google Looker Studio. Ele foi elaborado com base na análise exploratória de dados e no modelo final do dashboard, visando garantir a consistência da solução de Business Intelligence. O objetivo é fornecer uma referência clara para a estrutura das fontes de dados, campos calculados, visualizações e configurações do dashboard.
  
    
# Fontes de Dados  
  
O dashboard é construído a partir de dois conjuntos de dados alocados no Google Sheets.
  
* Tabela *colheita*  
  
|id	|dia		|id_partilha	|cultivo	|variedade	|quantidade	|tipo_cultivo	|mes		|total_partilha	|
|:------|--------------:|--------------:|--------------:|--------------:|--------------:|--------------:|--------------:|--------------:|
|1	|01/02/2020	|1		|batata_doce	|branca		|34,000		|tuberculo	|01/02/2020	|89,90		|
|2	|01/02/2020	|1		|abacate	|padrao		|20,000		|fruta		|01/02/2020	|89,90		|
|3	|01/02/2020	|1		|mamao		|formosa	|14,500		|fruta		|01/02/2020	|89,90		|
|4	|01/02/2020	|1		|abobora	|mini_moranga	|6,000		|legume		|01/02/2020	|89,90		|
|5	|01/02/2020	|1		|rucula		|padrao		|5,500		|hortalica	|01/02/2020	|89,90		|
  
  
* Tabela *coagricultores*  
  
|id_coagricultor|nome_coagricultor	|qt_cota	|entrada	|saida		|permanencia	|status		|
|:--------------|----------------------:|--------------:|--------------:|--------------:|--------------:|--------------:|
|1		|Prozaria		|9,00		|01/02/2020	|01/04/2020	|2,00		|INATIVO	|
|2		|Vitor e Annie		|1,00		|01/02/2020	|01/07/2022	|29,00		|INATIVO	|
|3		|Henrique e Denise	|1,00		|01/02/2020	|01/10/2020	|8,00		|INATIVO	|
|4		|Luana			|1,00		|01/02/2020	|01/07/2020	|5,00		|INATIVO	|
|5		|Carolzinha		|1,00		|01/02/2020	|01/12/2020	|10,00		|INATIVO	|
  
* Tabela *coagricultores_expandido*  
  
Para desenvolver o dashboard também é necessário uma tabela expandida dos coagricultores para cálculos mensais. Para criar e atualizar automaticamente a nova tabela deve ser utilizado o script [Expandir Coagricultores](#script-expandir-coagricultores) anexado ao final do documento.  

|id_coagricultor	|nome	|mes_ref	|qt_cota	|valot_qt	|  
|:----------------------|-------|---------------|---------------|---------------|
|1	|Prozaria	|01/02/2020	|9	|180	|
|1	|Prozaria	|01/03/2020	|9	|180	|
|1	|Prozaria	|01/04/2020	|9	|180	|
|2	|Vitor e Annie	|01/02/2020	|1	|180 	|
|2	|Vitor e Annie	|01/03/2020	|1	|180	|
 
  
# Estrutura do Dashboard e Visualizações  
  
O dashboard é composto por duas páginas principais: Desempenho da Produção e Análise da Produção.  
  
## Página 1: Desempenho Geral  
  
Esta página oferece uma visão executiva dos principais indicadores de produção, faturamento e engajamento dos coagricultores, com foco em tendências e comparações.  
  
### Filtro  
  
Controle por período para permitir que o usuário selecione o intervalo de datas.  
  
### Scorecards com KPIs  
  
Scorecards organizados em grupos de produção, diversidade, faturamento e coagricultores. Todos com o indicador de variação (%) da métrica com o período anterior.  
  
* **Produção Total**  
	* Fonte de dados: colheita
	* Métrica: quantidade  sd
	* Agregação: Soma (SUM)  
  
* **Colheitas no Período**  
	* Fonte de dados: colheita
	* Métrica: id_partilha  
	* Agregação: Contar diferentes (CTD)  
  
* **Média por Colheita**
	* Fonte de dados: colheita
	* Campo calculado: "Média kg por colheita"  
	* Fórmula: SUM(quantidade) / COUNT_DISTINCT(id_partilha)
	* Agregação: automático. 
  
* **Diversidade Total** 
  	* Fonte de dados: colheita 
  	* Campo calculado: "Cultivo + variedade"
	* Fórmula: CONCAT(cultivo, " ", variedade)
	* Agregação: automático
  
* **Média por colheita** 
	* Fonte de dados: colheita
	* Campo calculado: "Média variedade por colheita"
	* Fórmula: COUNT_DISTINCT(CONCAT(CAST(id_partilha AS TEXT), '-', variedade)) / COUNT_DISTINCT(id_partilha)
  	* Agregação: automático.  
  
* **Faturamento Total** 
	* Fonte de dados: coagricultores - coagricultores_expandido
	* Campo calculado "Faturamento"
	* Fórmula: qt_cota * valor_cota
	* Agregação: Soma (SUM)  
  
* **Faturamento Médio Mensal**  
	* Fonte de dados: coagricultores - coagricultores_expandido  
	* Campo calculado: "Faturamento mensal"  
	* Fórmula: SUM(faturamento) / COUNT_DISTINCT(mes_ref)
	* Agregação: automático  
  
* **Coagricultores Ativos**  
	* Fonte de dados: coagricultores - coagricultores_expandido
	* Métrica: id_coagricultor  
	* Agregação: Contar diferentes (CTD)  
  
* **Média de Permanência**
	* Fonte de dados: coagricultores - coagricultores_expandido
	* Campo calculado: Média permanência  
	* Fórmula: COUNT_DISTINCT(CONCAT(CAST(id_coagricultor AS TEXT), FORMAT_DATETIME( '%Y-%m',mes_ref ) ) ) / COUNT_DISTINCT(id_coagricultor)
	* Agregação: automático
  
### Visualizações Principais  
  
* **Tendência da Produção no Período**  
	* Tipo: Gráfico de Série Temporal
	* Fonte de Dados: colheita
	* Dimensão: dia 
	* Métrica: quantidade (renomeado para "Total colhido (kg)"
	* Agregação: soma (SUM)
	* Métrica de Comparação: Produção Total Kg (para Ano anterior)
	* Linha de tendência: linear
  
* **Tendência da Diversidade de Cultivo**
	* Tipo: Gráfico de Série Temporal
	* Fonte de Dados: colheita
	* Dimensão: dia
	* Métrica: cultivo (renomeado para "Diversidade de cultivo")
	* Agregação: contar diferentes (CTD)
	* Métrica de Comparação: Diversidade Total (Variedades) (para Ano anterior)
  
* **Maior Colheita / Menor Colheita**
	* Tipo: Scorecards
	* Fonte de Dados: colheita
	* Cards para Maior Colheita:
		* Dia da colheita
			* Dimensão: dia (renomeado "Dia da colheita")
			* Classificar: DESC(MAX(total_partilha))
		* ID da Colheita
			* Dimensão: id_partilha (renomeado "ID")
			* Classificar: DESC(MAX(total_partilha))
		* Total Colhido
			* Métrica: total_partilha
			* Agregação: Máximo (MAX)
	* Cards para Menor Colheita:  
		* Dia da colheita
			* Dimensão: dia (renomeado "Dia da colheita")
			* Classificar: ASC(MIN(total_partilha))
		* ID da Colheita
			* Dimensão: id_partilha (renomeado "ID")
			* Classificar: ASC(MIN(total_partilha))
		* Total Colhido
			* Dimensão: total_partilha
			* Classificar: ASC(MIN(total_partilha))
  
* **Faturamento por Mês (R$)**
	* Tipo: Gráfico de Barras
	* Fonte de Dados: coagricultores - coagricultores_expandido
	* Dimensão: mes_ref
	* Métrica: faturamento (renomeado "Faturamento (R$)")
	* Agregação: soma (SUM)
	
* **Coagricultores Ativos ao Longo do Tempo**
	* Tipo: Gráfico de Série Temporal
	* Fonte de Dados: coagricultores - coagricultores_expandido
	* Dimensão: mes_ref
	* Métrica: id_coagricultores (renomeado "Coagricultores Ativos"
	* Agregação: contar diferentes (CTD)
	* Métrica de Comparação: Coagricultores Ativos (para Ano anterior).


## Página 2: Análise da Produção  
  
Esta página aprofunda a análise da produção, focando na composição por tipo de cultivo e no detalhamento por colheita e variedade.  
  


### Filtros  
  
Filtros de Dimensão do tipo lista suspensa com os seguintes campos: Ano colheita, Mês colheita, Nº da Partilha, Tipo de Cultivo, Cultivo.


### Scorecards com KPIs
  
Reutilizar os seguintes scorecards já criados:
* Produção Total
* Colheitas no período
* Média por colheita
* Diversidade Total
* Média por colheita
  
### Visualizações detalhadas

* **Participação na Partilha (%) por ID da Colheita e Cultivo**
	* Tipo: Gráfico de Barras Empilhadas
	* Fonte de Dados: colheita
	* Dimensão: id_partilha (renomeado "ID da Colheita")
	* Dimensão de Detalhamento: cultivo 
	* Métrica: quantidade (renomeada para "Participação na Partilha (%)")
	* Agregação: soma (SUM)

* **Total de Cada Cultivo por Colheita**
	* Tipo: Gráfico de Área
	* Fonte de Dados: colheita
	* Dimensão: dia
	* Detalhar: dia
	* Dimensão de Detalhamento: Cultivo + variedade
	* Métrica: quantidade (renomeado para "Total por colheita (kg)")
	* Agregação: soma (SUM)
	* Classificação da dimensão detalhada: DESC(quantidade)

* **Total Colhido (Kg) por Tipo de Cultivo**
	* Tipo: Gráfico de Barras
	* Fonte de Dados: colheita
	* Dimensão: tipo_cultivo, cultivo e Cultivo + variedade
	* Nível de detalhamento padrão: tipo_cultivo
	* Dimensão detalhada: tipo_cultivo 
	* Métrica: quantidade (renomeado para "Total colhido (kg)")
	* Agregação: soma (SUM)
	* Classificação: DESC(SUM(quantidade))

* **Tabela de Resumo**
	* Tipo: Tabela Dinâmica com Mapa de Calor
	* Fonte de Dados: colheita
	* Dimensões da linha: cultivo, variedade.
	* Métricas:
		* Total colhido (kg) - SUM(quantidade)
		* Colheitas no Período - CT(quantidade)
		* Média por Colheita (Kg)
		* Maior Colheita - MAX(quantidade)
		* Menor Coheita - MIN(quantidade)
	* Classificação: DESC(SUM(quantidade))
 
# Conclusão

Este documento serve como um roteiro para a criação e manutenção do dashboard "Mãeã Natureza: Desempenho da Produção" no Looker Studio. O painel visa ser uma ferramenta de BI robusta, precisa e fácil de usar, capaz de fornecer insights valiosos para o Sítio Mãeã Natureza.

  
  
# Anexo  
  
## Script Expandir Coagricultores  
  
O script pode ser executado pelo Apps Script do Google.


	function expandirCoagricultores() {
	  const sheetFonte = SpreadsheetApp.getActive().getSheetByName("coagricultores");
	  const sheetCalendario = SpreadsheetApp.getActive().getSheetByName("calendario_mensal");
	  const sheetDestino = SpreadsheetApp.getActive().getSheetByName("coagricultores_expandido");
	  
	  // Pega todos os dados da planilha fonte
	  const dadosCoagri = sheetFonte.getDataRange().getValues();
	  
	  // Define a data de hoje (sem considerar a hora, para comparação de mês completo)
	  // Criamos uma data no formato AAAA-MM-01 para comparar apenas o mês e o ano
	  const hoje = new Date();
	  const hojePrimeiroDiaDoMes = new Date(hoje.getFullYear(), hoje.getMonth(), 1); // Ex: 2025-07-01
	  
	  // Pega apenas a coluna A do calendário, remove vazios e FILTRA os meses <= HOJE
	  const dadosMeses = sheetCalendario.getRange("A2:A").getValues().flat().filter(String).filter(mes_str => {
	    const mesCalendario = new Date(mes_str);
	    // Compara o primeiro dia do mês do calendário com o primeiro dia do mês atual
	    return mesCalendario <= hojePrimeiroDiaDoMes;
	  });
	  
	  const cabecalho = ["id_coagricultor", "nome", "mes_ref", "qt_cota", "valor_cota"];
	  let resultado = [cabecalho];
	
	  // Itera sobre os dados dos coagricultores, começando da segunda linha (índice 1) para pular o cabeçalho
	  for (let i = 1; i < dadosCoagri.length; i++) {
	    const [id, nome, qt_cota_original, entrada, saida, permanencia, valor_cota_ignorado] = dadosCoagri[i];
	    
	    // Converte as datas de entrada e saída para objetos Date
	    const dataEntrada = new Date(entrada);
	    // Se 'saida' estiver vazio, assume uma data futura bem distante
	    const dataSaida = saida ? new Date(saida) : new Date("2100-01-01"); // Usar uma data bem distante como 2100-01-01
	    
	    // Itera sobre cada mês de referência do calendário (que já estão pré-filtrados)
	    dadosMeses.forEach(mes_str => {
	      const mes = new Date(mes_str);
	      // Verifica se o mês de referência está dentro do período de permanência do coagricultor
	      if (mes >= dataEntrada && mes <= dataSaida) {
	        // Define o valor da cota com base na data do mês de referência
	        const valor_cota = mes < new Date("2025-01-01") ? 180 : 220; // Manter sua lógica de valor_cota
	        // Adiciona a linha expandida ao array de resultado
	        resultado.push([id, nome, mes_str, qt_cota_original, valor_cota]);
	      }
	    });
	  }
	
	  // Limpa o conteúdo da planilha de destino
	  sheetDestino.clearContents();
	  // Escreve o array de resultado na planilha de destino
	  sheetDestino.getRange(1, 1, resultado.length, resultado[0].length).setValues(resultado);
	}



