# 📊 Análise de Vendas com SQLite

A base de dados utilizada neste projeto é fictícia, elaborada exclusivamente para fins de portfólio. Ela simula o histórico de vendas de uma empresa do segmento de materiais de construção.

Este projeto tem por objetivo compreender o desempenho comercial, identificar produtos e categorias de maior impacto, avaliar a eficiência das estratégias comerciais, apoiar decisões de estoque e precificação, prever tendências e planejar metas futuras.

---

## 🧱 Estrutura da Base de Dados

### 🗃️ Tabela: `Vendas`
```sql
CREATE TABLE "Vendas" (
  "id_venda" INTEGER,
  "id_produto" INTEGER,
  "id_vendedor" INTEGER,
  "categoria" TEXT,
  "descricao_produto" TEXT,
  "data_venda" TEXT,
  "quantidade_vendida" INTEGER,
  "valor_unitario" DECIMAL(10,2),
  "valor_total_venda" DECIMAL(10,2)
);
```

### 👩‍💼 Tabela: `Vendedores`
```sql
CREATE TABLE "Vendedores" (
  "id_vendedor" INTEGER
  "nome" TEXT
  "unidade" TEXT,
);
```

---

## 📊 Consultas

```sql
-- Evolução do faturamento por ano
CREATE VIEW ViewEvolucaoAno AS
SELECT 
    strftime('%Y', data_venda) AS Ano, 
    SUM(valor_total_venda) AS Valor_total_vendido
FROM Vendas
GROUP BY Ano
ORDER BY Ano;
```
---

```sql
-- Evolução do faturamento por trimestre por ano
CREATE VIEW ViewEvolucaoFatTri AS
SELECT strftime('%Y', data_venda) AS Ano, 
	   CASE
       	  WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 1 AND 3 THEN '1ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 4 AND 6 THEN '2ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 7 AND 9 THEN '3ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 10 AND 12 THEN '4ºTRI'
      END AS Trimestre, 
      ROUND(SUM(valor_total_venda),2) AS Valor_total_vendido
FROM Vendas
GROUP BY Ano, Trimestre;
```
---

```sql
-- Evolução do faturamento por trimestre, visualizando somente um ano
SELECT strftime('%Y', data_venda) AS Ano, 
	   CASE
       	  WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 1 AND 3 THEN '1ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 4 AND 6 THEN '2ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 7 AND 9 THEN '3ºTRI'
          WHEN CAST (strftime('%m', data_venda) AS INTEGER) BETWEEN 10 AND 12 THEN '4ºTRI'
      END AS Trimestre, 
      ROUND(SUM(valor_total_venda),2) AS Valor_total_vendido
FROM Vendas
WHERE strftime('%Y', data_venda) = '2024'
GROUP BY Ano, Trimestre;
```
---

```sql
-- Evolução do faturamento por ano/mes
SELECT strftime('%Y/%m', data_venda) AS "Ano/Mes", ROUND(SUM(valor_total_venda), 2) AS Valor_total, 
ROUND((SUM(valor_total_venda) * 100.0)/(SELECT SUM(valor_total_venda) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas
GROUP BY "Ano/Mes"
ORDER BY "Ano/Mes" DESC;
```
---

```sql
-- Evolucao da venda quantidade por ano/mes
SELECT strftime('%Y/%m', data_venda) AS "Ano/Mes", SUM(quantidade_vendida) AS Qtd_total_vendida
FROM Vendas
GROUP BY "Ano/Mes"
ORDER BY "Ano/Mes" DESC;
```
---

```sql
-- Ranking de vendas - top 5 produtos mais vendidos (quantidade)
SELECT 
  descricao_produto,
  ROUND(SUM(quantidade_vendida),2) AS Qtd_total_vendida,
  ROUND((SUM(quantidade_vendida) * 100.0)/(SELECT SUM(quantidade_vendida) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas
GROUP BY descricao_produto
ORDER BY Qtd_total_vendida DESC
LIMIT 5;
```
---

```sql
-- Ranking de vendas - top 5 produtos (faturamento)
SELECT 
  descricao_produto,
  ROUND(SUM(valor_total_venda),2) AS Valor_total_vendido,
  ROUND((SUM(valor_total_venda) * 100.0)/(SELECT SUM(valor_total_venda) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas
GROUP BY descricao_produto
ORDER BY Valor_total_vendido DESC
LIMIT 5;
```
---

```sql
-- Ranking de vendas - por categoria (quantidade)
SELECT 
  categoria,
  SUM(quantidade_vendida) AS Qtd_total_vendida,
  ROUND((SUM(quantidade_vendida)*100.0)/(SELECT SUM(quantidade_vendida) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas
GROUP BY categoria
ORDER BY Qtd_total_vendida DESC;
```
---

```sql
-- Ranking de vendas - por categoria (faturamento)
SELECT 
  categoria,
  SUM(valor_total_venda) AS Valor_total_vendido,
  ROUND((SUM(valor_total_venda)*100.0)/(SELECT SUM(valor_total_venda) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas
GROUP BY categoria
ORDER BY Valor_total_vendido DESC;
```
---

```sql
-- Ranking de vendas por vendedor (quantidade)
CREATE VIEW ViewVendasPorVendedor AS
SELECT ve.id_vendedor, v.nome, SUM(quantidade_vendida) AS Qtd_total_vendida,
ROUND((SUM(quantidade_vendida)*100.0)/(SELECT SUM(quantidade_vendida) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas ve
INNER JOIN Vendedores v
ON ve.id_vendedor = v.id_vendedor
GROUP BY v.id_vendedor
ORDER BY Qtd_total_vendida DESC;
```
---

```sql
-- Ranking de vendas por vendedor (valor)
SELECT ve.id_vendedor, v.nome, SUM(valor_total_venda) AS Valor_total,
ROUND((SUM(valor_total_venda)*100.0)/(SELECT SUM(valor_total_venda) FROM Vendas), 2) AS Percentual_sobre_total
FROM Vendas ve
INNER JOIN Vendedores v
ON ve.id_vendedor = v.id_vendedor
GROUP BY v.id_vendedor
ORDER BY Valor_total DESC;
```
---

```sql
-- Ticket médio por venda 
-- Fórmula padrão: Faturamento total/ número de vendas
-- Aqui não usei o distinct, pois estou fazendo o ticket médio por venda e não por cliente

CREATE VIEW ViewTicketMedio AS
SELECT ROUND(SUM(quantidade_vendida * valor_unitario) 
       / COUNT(id_venda),2) AS Ticket_medio_por_venda
FROM Vendas;

```
---

📌 **Autor:** *Marianna Fernandes Salviano*  
🧩 **SGBD:** SQLite  
🎯 **Propósito:** Projeto analítico para portfólio e processos seletivos.
