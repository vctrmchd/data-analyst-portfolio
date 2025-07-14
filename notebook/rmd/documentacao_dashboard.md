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


  
# Estrutura do Dashboard e Visualizações  
  
O dashboard é composto por duas páginas principais: Desempenho da Produção e Análise da Produção.  
  
## Página 1: Desempenho Geral  
  
Esta página oferece uma visão executiva dos principais indicadores de produção, faturamento e engajamento dos coagricultores, com foco em tendências e comparações.  
  
### Filtro  
  
Controle por período para permitir que o usuário selecione o intervalo de datas.  
  
### Scorecards com KPIs  
  
Scorecards organizados em grupos de produção, diversidade, faturamento e coagricultores. Todos com o indicador de variação (%) da métrica com o período anterior.  
  
* **Produção Total**  
	*Fonte de dados: colheita
	*Métrica: quantidade  
	*Agregação: Soma (SUM)  
  
* **Colheitas no Período**  
	**Fonte de dados: colheita
	**Métrica: id_partilha  
	**Agregação: Contar diferentes (CTD)  
  
* **Média por Colheita**
	**Fonte de dados: colheita
	**Campo calculado: "Média kg por colheita"  
	**Fórmula: SUM(quantidade) / COUNT_DISTINCT(id_partilha)
	**Agregação: automático. 
  
* **Diversidade Total** 
	**Fonte de dados: colheita 
	**Campo calculado: "Cultivo + variedade"
	**Fórmula: CONCAT(cultivo, " ", variedade)
	**Agregação: automático
  
* **Média por colheita** 
	**Fonte de dados: colheita
	**Campo calculado: "Média variedade por colheita"
	**Fórmula: COUNT_DISTINCT(CONCAT(CAST(id_partilha AS TEXT), '-', variedade)) / COUNT_DISTINCT(id_partilha)
	**Agregação: automático.  
  
* **Faturamento Total** 
	**Fonte de dados: coagricultores - coagricultores_expandido
	**Campo calculado "Faturamento"
	**Fórmula: qt_cota * valor_cota
	**Agregação: Soma (SUM)  
  
* **Faturamento Médio Mensal**  
	**Fonte de dados: coagricultores - coagricultores_expandido  
	**Campo calculado: "Faturamento mensal"  
	**Fórmula: SUM(faturamento) / COUNT_DISTINCT(mes_ref)
	**Agregação: automático  
  
* **Coagricultores Ativos**  
	**Fonte de dados: coagricultores - coagricultores_expandido
	**Métrica: id_coagricultor  
	**Agregação: Contar diferentes (CTD)  
  
* **Média de Permanência**
	**Fonte de dados: coagricultores - coagricultores_expandido
	**Campo calculado: Média permanência  
	**Fórmula: COUNT_DISTINCT(CONCAT(CAST(id_coagricultor AS TEXT), FORMAT_DATETIME( '%Y-%m',mes_ref ) ) ) / COUNT_DISTINCT(id_coagricultor)
	** Agregação: automático
  
  
  
  
  

# Scrip Expandir Coagricultores  
  
O script pode ser executado pelo Apps Script do Google.

´´´r
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
´´´
