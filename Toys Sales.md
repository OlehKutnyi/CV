# Stores sales
The purpose of this project is to analyse the sales of toys and similar products in a big store network. It is created to demonstrate and practice my SQL and Tableau skills. The project is based on the dataset Maven Toys and their sales in different shops. The Schema consists of four tables: **Products** (product id, product name, product category, product cost product price); **Inventory** (ProductID, StoreID, stock on hand); **Stores** (StoreID, store name, store city, store location, store open date); **Sales** (SaleID, date, StoreID, ProductID, units)

### Products
| Product ID | Product name    | Product category | Product cost | Product price |
|------------|-----------------|------------------|--------------|---------------|
| 1          | Action Figure   | Toys             |  9.99        | 15.99         |
| 2          | Animal Figures  | Art & Crafts     |  1.99        | 3.99          |
| 3          | Barrel O' Slime | Games            |  7.99        | 9.99          |

### Inventory
| Store ID  | Product ID | stock on hand |
|-----------|------------|---------------|
| 1         | 1          | 27            |
| 1         | 2          | 0             |
| 1         | 3          | 32            |

### Stores
| Store ID |    Store name             | Store city   | Store location | Store open date |
|----------|---------------------------|--------------|----------------|-----------------|
| 1        | Maven Toys Guadalajara 1  | Guadalajara  | Residential    | 1992-09-18      |
| 2        | Maven Toys Monterrey 1    | Monterrey    | Residential    | 1995-04-27      |
| 3        | Maven Toys Guadalajara 2  | Guadalajara  | Commercial     | 1999-12-27      |

### Sales
| Sale ID | Date       | Store ID | Product ID | Units |
|---------|------------|----------|------------|-------|
| 1       | 2022-01-01 | 24       | 4          | 1     |
| 2       | 2022-01-01 | 28       | 1          | 2     |
| 3       | 2022-01-01 | 6        | 8          | 1     |


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

<img width="480" alt="Знімок екрана 2024-01-18 о 18 09 43" src="https://github.com/OlehKutnyi/CV/assets/150731232/681007fe-f929-4f51-b218-7b87f4063eee">

Then I found the category with the highest average margin

```SQL
SELECT
    Products.product_category,
    ROUND(AVG(product_price) - AVG(product_cost),2) AS margin
FROM Products
GROUP BY product_category
ORDER BY margin DESC;
```
<img width="366" alt="Знімок екрана 2024-01-18 о 18 12 33" src="https://github.com/OlehKutnyi/CV/assets/150731232/5a97bf9c-62d2-46e8-b954-5e162db386ac">

Looks like electronics should be very profitable as it has both high sales and the highest margin.

Cities with high sales

```SQL
SELECT
    Stores.store_city,
    SUM(Sales.units) AS city_sales
FROM Stores
JOIN Sales ON Stores.StoreID = Sales.StoreID
GROUP BY Stores.store_city
ORDER BY city_sales DESC
LIMIT 10;
```

<img width="369" alt="Знімок екрана 2024-01-18 о 18 14 50" src="https://github.com/OlehKutnyi/CV/assets/150731232/a6f1df42-f835-4f50-8de6-ea99f106fd17">

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
**Profit = 373368.00**

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
<img width="367" alt="Знімок екрана 2024-01-18 о 18 19 13" src="https://github.com/OlehKutnyi/CV/assets/150731232/c43451d7-780d-47b3-b4ed-40cd33a35276">

I've also calculated the average revenue per store

```SQL
SELECT
    Stores.StoreID,
    Stores.Store_name,
    CONCAT(Stores.store_location, " ", Stores.store_city) as store_full_location,
    SUM(Sales.units * Products.product_price) / DATEDIFF(MAX(Sales.date), MIN(Sales.date)) * 30 as average_revenue_per_month
FROM Stores
RIGHT JOIN Sales ON Stores.StoreID = Sales.StoreID
LEFT JOIN Products ON Sales.ProductID = Products.ProductID
GROUP BY Stores.StoreID;
```
<img width="1440" alt="Знімок екрана 2024-02-05 о 21 58 50" src="https://github.com/OlehKutnyi/CV/assets/150731232/88b37d82-1e9d-4dec-a1dc-e448cae0a425">


