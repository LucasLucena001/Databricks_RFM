1) Carregamento dos dados

O notebook acessa um arquivo chamado “Online Retail.csv” dentro do Databricks (usando Spark) e informa que o separador de colunas é ;.

Em seguida, converte esse conjunto de dados do Spark para pandas (uma forma mais simples de manipular dados em memória).

Faz checagens iniciais: tamanho da tabela, tipos das colunas e uma amostra de linhas, só para garantir que leu tudo certo.

2) Limpeza dos dados (garantir qualidade)

O objetivo aqui é manter apenas vendas válidas:

Remove linhas sem identificador de cliente (CustomerID) — sem este dado não dá para analisar o comportamento do cliente.

Exclui notas de cancelamento, identificadas por InvoiceNo começando com “C”.

Mantém apenas quantidades positivas (Quantity > 0) e preços positivos (UnitPrice > 0).

3) Valor total por linha

Cria uma coluna TotalPrice multiplicando Quantidade × Preço unitário.
Isso representa quanto foi gasto em cada linha do pedido.

4) Data de referência

Define a data de referência como a última data de compra encontrada no conjunto de dados (após a limpeza).
Ela serve para calcular “há quantos dias o cliente comprou pela última vez”.

5) Cálculo do RFM (por cliente)

Agrupa os dados por cliente e calcula três medidas:

Recency (Recência): quantos dias se passaram desde a última compra do cliente até a data de referência.
Interpretação: quanto menor o número, mais recente o cliente comprou.

Frequency (Frequência): quantas compras diferentes o cliente fez (contando notas/InvoiceNo distintos).
Interpretação: quanto maior, mais vezes ele comprou.

Monetary (Monetário): quanto de dinheiro o cliente gastou no total (soma de TotalPrice).
Interpretação: quanto maior, maior seu valor financeiro.

6) Transformar R, F e M em “escores” de 1 a 5

Para facilitar a comparação entre clientes, cada métrica vira um escala de 1 (pior) a 5 (melhor):

R_Score: invertido (clientes mais recentes ganham nota maior).

F_Score: baseado na frequência (quem comprou mais vezes tende a nota maior).

M_Score: baseado no valor gasto (quem gastou mais tende a nota maior).

Também são criadas colunas de conveniência:

RFM_Score: a string juntando as três notas (ex.: “545”).

RFM_Sum: a soma numérica das três notas (ex.: 5+4+5 = 14).

Observação importante: essas duas colunas são opcionais. Elas ajudam em filtros e ordenações, mas não são obrigatórias para construir gráficos ou segmentações. Não há problema em não usá-las no dashboard.

7) Classificação em segmentos (rótulos de grupos)

Com base nas notas R, F e M, o notebook atribui um segmento a cada cliente. As regras (simplificadas) são:

Campeões: notas altas nas três frentes (R, F e M) — clientes muito valiosos e ativos.

Leais: compram com frequência e recentemente, mesmo que o gasto não seja sempre altíssimo.

Promissores: compraram recentemente, ainda não compram com frequência, mas já mostram bom gasto.

Em risco: gastam e/ou compram bem, mas faz tempo que não compram.

Hibernando (alto gasto): gastaram muito no passado, mas estão inativos e não eram muito frequentes.

Novos/baixo valor: clientes recentes, mas com pouca frequência e pouco gasto (ainda).

Atenção: casos que não se encaixam tão claramente nas regras acima.

Esses nomes/rótulos ajudam a pensar ações práticas:
• “Campeões/Leais”: programas de fidelidade, acesso antecipado a lançamentos.
• “Em risco/Hibernando”: campanhas de reativação com oferta.
• “Promissores/Novos”: nutrição e incentivo para aumentar frequência/valor.

8) Tabelas de resumo para análise

O notebook gera tabelas úteis para o dashboard:

Distribuição por segmento: quantos clientes em cada grupo.

Ticket médio por segmento: gasto médio (Monetary) por grupo.

Top 20 clientes por gasto: quem mais gastou (útil para ações VIP).

9) Enriquecimento com país e data da última compra

Ele prepara um cruzamento para pegar, para cada cliente, o país e a data da última compra (ex.: “Country”, “LastPurchaseAt”) a partir do registro mais recente do próprio cliente, e anexa isso ao resultado RFM (criando algo como rfm_country).
Isso permite análises por país e por momento da última compra.

10) Gravação da tabela final no Databricks

Converte o resultado final em Spark DataFrame e salva como uma tabela Delta no catálogo:
analytics.marketing.customers_rfm (criando o catálogo e o schema, se necessário).
Assim, essa tabela pode ser consumida por dashboards e outras ferramentas de análise.
