# Análise do dataset do E-commerce brasileiro Olist - Linguagem SQL

O dataset utilizado neste projeto de análise foi disponibilizado no Kaggle, pela Olist, loja de departamentos de e-commerce brasileira. A empresa conecta pequenas empresas de todo o Brasil a grandes marketplaces através da Loja Olist.

O conjunto de dados contém informações de 100 mil pedidos de 2016 a 2018 feitos no Brasil. Seus recursos permitem visualizar um pedido em várias dimensões: desde o status do pedido, preço, desempenho de pagamento e frete até a localização do cliente, atributos do produto e, finalmente, avaliações escritas pelos clientes. Também lançamos um conjunto de dados de geolocalização que relaciona os códigos postais brasileiros às coordenadas lat / lng.

**Dataset**

O dataset está disponível no Kaggle através do link: https://www.kaggle.com/olistbr/brazilian-ecommerce

**SGBD**

Foi utilizado o SQL Server para importar o dataset e fazer as análises de negócio

**Estrutura do Dataset**

![Olist_Schema](https://user-images.githubusercontent.com/64870434/94340486-80f85a80-ffd8-11ea-9c08-66978ca8e888.png)

**Tópicos do projeto**

- **1. Pedidos**: Análise dos pedidos
- **2. Vendas**: Análise das vendas por categoria e comprativo anual
- **3. Sellers**: Análise do número de sellers por estado e desempenho dos melhores/piores

**1. Selecionando BD**

```
USE OLIST;
```

**1.1. Verificando colunas de uma tabela**

```
SP_HELP olist_geolocation_dataset;
```

**1.2. Número de cidades por estado que tiveram pedidos entre 2016 E 2018**

```
SELECT geolocation_state AS ESTADO, COUNT(DISTINCT geolocation_city) AS NUMERO_CIDADES_COM_PEDIDO
FROM olist_geolocation_dataset
GROUP BY geolocation_state
ORDER BY 2 DESC;
```

**1.3. Análise com ID do pedido e produtos com valor do produto em si, valor do frete e valor final da venda - utilização de JOIN de 3 tabelas**

```
SELECT A.order_id AS NUM_PEDIDO, A.product_id AS NUM_PRODUTO, A.price AS VALOR_PRODUTO, A.freight_value AS VALOR_FRETE,
B.payment_value AS TOTAL,
C.product_category_name AS CATEGORIA
FROM olist_order_items_dataset A
INNER JOIN olist_order_payments_dataset B
ON A.order_id = B.order_id
INNER JOIN olist_products_dataset C
ON A.product_id = C.product_id;
```

**2. Vendas por Categoria**

```
SELECT A.product_category_name AS CATEGORIA, SUM(B.price) AS VALOR_TOTAL
FROM olist_products_dataset A
INNER JOIN olist_order_items_dataset B
ON A.product_id = B.product_id
GROUP BY A.product_category_name
ORDER BY A.product_category_name;
```

**2.1. Análise ordenada por categoria com maior valor de vendas e identificação do valor total de produtos sem categoria definida**

```
SELECT 
CASE
WHEN A.product_category_name = '' THEN 'SEM CATEGORIA DEFINIDA'
ELSE A.product_category_name
END
AS CATEGORIA,
SUM(B.price) AS VALOR_TOTAL
FROM olist_products_dataset A
INNER JOIN olist_order_items_dataset B
ON A.product_id = B.product_id
GROUP BY A.product_category_name
ORDER BY SUM(B.price) DESC;
```

**2.2. Comparativo do valor de vendas total com cada ano - uso de subquerys**

```
SELECT SUM(P.payment_value) AS 'VENDAS_TOTAL',
			(SELECT SUM(P.payment_value)
			FROM olist_order_payments_dataset P
			INNER JOIN olist_orders_dataset D
			ON P.order_id = D.order_id
			WHERE D.order_approved_at BETWEEN '2016-01-01 00:00:01' AND '2017-12-31 23:59:59') AS '2016',
				(SELECT SUM(P.payment_value)
				FROM olist_order_payments_dataset P
				INNER JOIN olist_orders_dataset D
				ON P.order_id = D.order_id
				WHERE D.order_approved_at BETWEEN '2017-01-01 00:00:01' AND '2017-12-31 23:59:59') AS '2017',
						(SELECT SUM(P.payment_value)
						FROM olist_order_payments_dataset P
						INNER JOIN olist_orders_dataset D
						ON P.order_id = D.order_id
						WHERE D.order_approved_at BETWEEN '2018-01-01 00:00:01' AND '2018-12-31 23:59:59') AS '2018'
FROM olist_order_payments_dataset P;
```

**2.3. Total de vendas nos 3 anos**

```
SELECT SUM(O.price) AS VALOR_TOTAL_PRODUTOS, SUM(O.freight_value) AS VALOR_FRETE, 
SUM(P.payment_value) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset O
INNER JOIN olist_order_payments_dataset P
ON O.order_id = P.order_id;
```

**2.4. Total de vendas por ano - separado, sem subquery**

**2016**

```
SELECT SUM(O.price) AS VALOR_PRODUTOS_VENDIDOS_2016, SUM(O.freight_value) AS VALOR_FRETE_2016, 
SUM(P.payment_value) AS TOTAL_VENDAS_2016
FROM olist_order_items_dataset O
INNER JOIN olist_order_payments_dataset P
ON O.order_id = P.order_id
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2016-01-01 00:00:01' AND '2016-12-31 23:59:59';
```

**2017**

```
SELECT SUM(O.price) AS VALOR_PRODUTOS_VENDIDOS_2017, SUM(O.freight_value) AS VALOR_FRETE_2017, 
SUM(P.payment_value) AS TOTAL_VENDAS_2017
FROM olist_order_items_dataset O
INNER JOIN olist_order_payments_dataset P
ON O.order_id = P.order_id
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2017-01-01 00:00:01' AND '2017-12-31 23:59:59';
```

**2018**

```
SELECT SUM(O.price) AS VALOR_PRODUTOS_VENDIDOS_2018, SUM(O.freight_value) AS VALOR_FRETE_2018, 
SUM(P.payment_value) AS TOTAL_VENDAS_2018
FROM olist_order_items_dataset O
INNER JOIN olist_order_payments_dataset P
ON O.order_id = P.order_id
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2018-01-01 00:00:01' AND '2018-12-31 23:59:59';
```

**3. Análise Sellers**

**3.1. Número de sellers por estado** 

```
SELECT seller_state AS ESTADO, COUNT(seller_id) AS NUMERO_SELLERS
FROM olist_sellers_dataset
GROUP BY seller_state
ORDER BY COUNT(seller_id) DESC;
```

**3.2. Vendas por Seller (vendedores)**

```
SELECT seller_id AS ID_SELLER, 
SUM(price*order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset
GROUP BY seller_id
ORDER BY SUM(price*order_item_id) DESC;
```

**3.3. Top performance - 10 seller com maior valor de vendas nos 3 anos analisados**

```
SELECT TOP 10 seller_id AS ID_SELLER, 
SUM(price*order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset
GROUP BY seller_id
ORDER BY SUM(price*order_item_id) DESC;
```

**3.4. Piores performances - 10 sellers que menos venderam nos 3 anos**

```
SELECT TOP 10 seller_id AS ID_SELLER, 
SUM(price*order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset
GROUP BY seller_id
ORDER BY SUM(price*order_item_id) ASC;
```

**3.5. 10 seller com maior volume de vendas em cada ano**

**2016**

```
SELECT TOP 10 O.seller_id AS ID_SELLER, 
SUM(O.price*O.order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset O
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2016-01-01 00:00:01' AND '2016-12-31 23:59:59'
GROUP BY O.seller_id
ORDER BY SUM(O.price*O.order_item_id) DESC;
```

**2017**

```
SELECT TOP 10 O.seller_id AS ID_SELLER, 
SUM(O.price*O.order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset O
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2017-01-01 00:00:01' AND '2017-12-31 23:59:59'
GROUP BY O.seller_id
ORDER BY SUM(O.price*O.order_item_id) DESC;
```

**2018**

```
SELECT TOP 10 O.seller_id AS ID_SELLER, 
SUM(O.price*O.order_item_id) AS VALOR_TOTAL_VENDAS
FROM olist_order_items_dataset O
INNER JOIN olist_orders_dataset D
ON O.order_id = D.order_id
WHERE D.order_approved_at BETWEEN '2018-01-01 00:00:01' AND '2018-12-31 23:59:59'
GROUP BY O.seller_id
ORDER BY SUM(O.price*O.order_item_id) DESC;
```
