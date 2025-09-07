# Análise RFM no Databricks

Este notebook implementa uma análise **RFM (Recency, Frequency, Monetary)** utilizando o Databricks como ambiente principal.  
A seguir, descrevo todas as etapas realizadas no processo.

---

## 1) Carregamento dos dados
- Leitura do arquivo **Online Retail.csv** dentro do Databricks, utilizando **Spark**, com separador `;`.
- Conversão do DataFrame de Spark para **pandas** para manipulação mais simples.
- Verificação inicial: tamanho da tabela, tipos de colunas e amostras de linhas para garantir que o carregamento ocorreu corretamente.

---

## 2) Limpeza dos dados (garantir qualidade)
Foram mantidas apenas as vendas válidas:
- Remoção de linhas sem **CustomerID** (sem identificador não é possível analisar comportamento).
- Exclusão de notas de cancelamento (**InvoiceNo** iniciando com "C").
- Filtro de transações com **quantidade e preço positivos**.

---

## 3) Valor total por linha
- Criação da coluna **`TotalPrice`** = `Quantity × UnitPrice`.  
- Representa quanto foi gasto em cada linha do pedido.

---

## 4) Data de referência
- Definição da **data de referência** como a última data de compra registrada (após a limpeza).  
- Essa data é usada para calcular a **recência** das compras dos clientes.

---

## 5) Cálculo do RFM (por cliente)
Agrupamento dos dados por cliente para cálculo das três dimensões:

- **Recency (Recência):** dias desde a última compra até a data de referência.  
  *Quanto menor, mais recente foi a compra.*
- **Frequency (Frequência):** número de compras distintas (InvoiceNo).  
  *Quanto maior, mais frequente é o cliente.*
- **Monetary (Monetário):** soma total gasta (TotalPrice).  
  *Quanto maior, mais valioso financeiramente.*

---

## 6) Transformação em escores (1 a 5)
Cada métrica foi transformada em uma escala de 1 a 5, facilitando a comparação:

- **R_Score:** invertido (clientes mais recentes = notas maiores).  
- **F_Score:** baseado na frequência (mais compras = nota maior).  
- **M_Score:** baseado no valor gasto (mais gasto = nota maior).  

Colunas adicionais:
- **`RFM_Score`**: string juntando as três notas (ex.: `"545"`).  
- **`RFM_Sum`**: soma numérica das três notas (ex.: `14`).  

> Observação: essas colunas são opcionais. Servem para facilitar ordenações e filtros, mas não são obrigatórias para gráficos ou segmentações.

---

## 7) Segmentação de clientes
Com base nos escores, cada cliente foi classificado em segmentos estratégicos:

- **Campeões:** notas altas em R, F e M.  
- **Leais:** compras frequentes e recentes, mesmo sem gasto altíssimo.  
- **Promissores:** compras recentes, mas ainda pouca frequência.  
- **Em risco:** costumavam gastar bem, mas não compram há muito tempo.  
- **Hibernando:** alto gasto no passado, hoje inativos.  
- **Novos / baixo valor:** clientes recentes, com pouca frequência e gasto.  
- **Atenção:** casos que não se encaixam claramente nos demais grupos.

### Ações práticas possíveis:
- **Campeões/Leais:** programas de fidelidade, pré-lançamentos exclusivos.  
- **Em risco/Hibernando:** campanhas de reativação.  
- **Promissores/Novos:** estímulos para aumentar frequência e gasto.

---

## 8) Tabelas de resumo
Foram criadas tabelas auxiliares para análises e dashboards:
- Distribuição de clientes por segmento.  
- Ticket médio por segmento.  
- Top 20 clientes por gasto total.  

---

## 9) Enriquecimento com país e última compra
- Inclusão de informações adicionais: **Country** e **LastPurchaseAt** para cada cliente, considerando o registro mais recente.  
- Permite análises por país e pelo momento da última compra.

---

## 10) Gravação da tabela final no Databricks
- Conversão do resultado final em DataFrame Spark.  
- Salvamento em formato **Delta Table** no catálogo:  
  **`analytics.marketing.customers_rfm`**.  

Essa tabela final está pronta para ser consumida por dashboards e análises de marketing.

---
