# Stores sales
The purpose of this project is to analyse the sales of toys and similar products in a big store network. It is created to demonstrate and practice my SQL and Tableau skills. The project is based on the dataset [Mexico Toys](https://www.kaggle.com/datasets/mysarahmadbhat/toy-sales) and their sales in different shops. The Schema consists of four tables: **Products** (product id, product name, product category, product cost product price); **Inventory** (ProductID, StoreID, stock on hand); **Stores** (StoreID, store name, store city, store location, store open date); **Sales** (SaleID, date, StoreID, ProductID, units)

### Products
| Product ID | Product name    | Product category | Product cost | Product price |
|------------|-----------------|------------------|--------------|---------------|
| 1          | Action Figure   | Toys             |  9.99        | 15.99         |
| 2          | Animal Figures  | Art & Crafts     |  1.99        | 3.99          |
| 3          | Barrel O' Slime | Games            |  7.99        | 9.99          |
| ...        | ...             | ...              | ...          | ...           |

### Inventory
| Store ID  | Product ID | stock on hand |
|-----------|------------|---------------|
| 1         | 1          | 27            |
| 1         | 2          | 0             |
| 1         | 3          | 32            |
| ...       | ...        | ...           |

### Stores
| Store ID |    Store name             | Store city   | Store location | Store open date |
|----------|---------------------------|--------------|----------------|-----------------|
| 1        | Maven Toys Guadalajara 1  | Guadalajara  | Residential    | 1992-09-18      |
| 2        | Maven Toys Monterrey 1    | Monterrey    | Residential    | 1995-04-27      |
| 3        | Maven Toys Guadalajara 2  | Guadalajara  | Commercial     | 1999-12-27      |
| ...      | ...                       | ...          | ...            | ...             |

### Sales
| Sale ID | Date       | Store ID | Product ID | Units |
|---------|------------|----------|------------|-------|
| 1       | 2022-01-01 | 24       | 4          | 1     |
| 2       | 2022-01-01 | 28       | 1          | 2     |
| 3       | 2022-01-01 | 6        | 8          | 1     |
| ...     | ...        | ...      | ...        | ...   |


## Querries

Firstly I decided to find sales by category. 

```SQL
SELECT
    Products.product_category,
    SUM(Sales.units) AS sales_per_category
FROM Products
RIGHT JOIN Sales ON Products.ProductID = Sales.ProductID
GROUP BY Products.product_category
ORDER BY SUM(Sales.units) DESC;
```

<img width="525" alt="Знімок екрана 2024-02-14 о 21 59 39" src="https://github.com/OlehKutnyi/CV/assets/150731232/e6cef752-7b8b-47ab-a54b-ed4261bd400e">


Then I found the category with the highest average margin

```SQL
SELECT
    Products.product_category,
    ROUND(AVG(product_price) - AVG(product_cost),2) AS margin
FROM Products
GROUP BY product_category
ORDER BY margin DESC;
```

<img width="368" alt="Знімок екрана 2024-02-14 о 22 00 15" src="https://github.com/OlehKutnyi/CV/assets/150731232/40c45191-a855-4b9d-a016-35cdfa360f3f">


Looks like electronics should be very profitable as it has both high sales and the highest margin.

Cities with high sales in comparison to the cities with lowest sales

```SQL
WITH city_sales AS (
    SELECT
        Stores.store_city,
        SUM(Sales.units) AS total_units_sold
    FROM Stores
    RIGHT JOIN Sales ON Stores.StoreID = Sales.StoreID
    GROUP BY Stores.store_city
)
, ranked_cities AS (
    SELECT
        store_city,
        total_units_sold,
        RANK() OVER (ORDER BY total_units_sold DESC) AS rank_high,
        RANK() OVER (ORDER BY total_units_sold ASC) AS rank_low
    FROM city_sales
)
SELECT
    rc_high.store_city AS best_cities,
    rc_high.total_units_sold AS city_sales_top,
    rc_low.store_city AS worst_cities,
    rc_low.total_units_sold AS city_sales_bottom
FROM ranked_cities AS rc_high
JOIN ranked_cities AS rc_low ON rc_high.rank_high = rc_low.rank_low
ORDER BY rc_low.total_units_sold ASC
LIMIT 10;
```

<img width="807" alt="Знімок екрана 2024-02-14 о 22 02 21" src="https://github.com/OlehKutnyi/CV/assets/150731232/7e7e43d8-fc51-4515-abed-10f7162a9e8e">


All time profit from all shops. 
```SQL
WITH product_margin AS (
    SELECT
        Products.ProductID,
        (Products.product_price - Products.product_cost) AS margin
    FROM Products
)
SELECT
    SUM(Sales.units * product_margin.margin) AS all_time_profit
FROM Sales
LEFT JOIN product_margin ON Sales.ProductID = product_margin.ProductID;
```
**Profit = 4014029.00**

Last month's sales of every shop
```SQL
SELECT
 Stores.StoreID,
 SUM(Sales.units) AS TotalUnitsSold
FROM Stores
RIGHT JOIN Sales ON Stores.StoreID = Sales.StoreID
WHERE Sales.Date BETWEEN (SELECT MAX(Date) - INTERVAL 30 DAY FROM Sales) AND (SELECT MAX(Date) FROM Sales)
GROUP BY Stores.StoreID
ORDER BY TotalUnitsSold DESC;
```

<img width="368" alt="Знімок екрана 2024-02-14 о 22 03 56" src="https://github.com/OlehKutnyi/CV/assets/150731232/67f41d96-c73b-4eba-a051-c8a864d0bd39">


I've also calculated the average revenue per store

```SQL
SELECT
    Stores.StoreID,
    Stores.Store_name,
    CONCAT(Stores.store_location, " ", Stores.store_city) as store_full_location,
    ROUND(SUM(Sales.units * Products.product_price) / DATEDIFF(MAX(Sales.date), MIN(Sales.date)) * 30 , 2) as average_revenue_per_month
FROM Stores
RIGHT JOIN Sales ON Stores.StoreID = Sales.StoreID
LEFT JOIN Products ON Sales.ProductID = Products.ProductID
GROUP BY Stores.StoreID;
```

<img width="1047" alt="Знімок екрана 2024-02-14 о 22 55 22" src="https://github.com/OlehKutnyi/CV/assets/150731232/db861e0a-2d51-441d-b1bf-84a083837841">


