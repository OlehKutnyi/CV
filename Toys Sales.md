# Stores sales
This project doesn't really have anything specific to discover. It is created to show some of my SQL and Tableau skills. The project is based on the dataset Maven Toys and their sales in different shops. The Schema consists of four tables: Products (product, product name, product category, product cost product price); Inventory (ProductID, StoreID, stock on hand); Stores (StoreID, store name, store city, store location, store open date); Sales(SaleID, date, StoreID, ProductID, units)

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

