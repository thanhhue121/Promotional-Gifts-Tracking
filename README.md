# 📊 TRACKING PROMOTIONAL GIFTS AND REVENUE IMPACT

## 📌 Overview
I want to share with you one of my daily tasks, which is tracking **promotional gifts** in terms of **cost, revenue uplift, and gift count**.  
The relevant data is spread across **multiple different databases** (mainly from`og_order` and `og_reward`), I will use SQL and Excel to extract the final results.

---
```sql
-- Step 1: Define sales order (SO) dataset with regional classification
WITH SO AS (
    SELECT DISTINCT 
        CASE 
            WHEN warehousecode LIKE 'CENHAN%' THEN 'Miền Bắc' 
            ELSE 'Miền Nam' 
        END AS Region,
        DATE(SO.orderdate) AS order_date, 
        SD.ordernumber,
        SD.productid,
        SD.productname,
        SD.productsku,
        SO.buyerphone, 
        SO.stationcode, 
        SUM(SD.subtotal) AS subtotal,
        SUM(SD.quantity) AS quantity, 
        COUNT(DISTINCT SO.buyerphone) AS unique_customers, 
        SUM(SD.subtotal) AS revenue 
    FROM og_order.v_salesorder_col AS SO
    LEFT JOIN og_order.v_salesorderdetail_col AS SD 
        ON SO.ordernumber = SD.ordernumber
    WHERE DATE(SO.orderdate) >= '2025-01-01' 
        AND orderstatus <> 13 
        AND warehousecode = 'CENHCM01'
    GROUP BY 1,2,3,4,5,6,7,8
),

--Step 2: Identify rewarded products (QT dataset)
QT AS (
    SELECT 
        A.*, 
        SO.productid AS QT_productid,
        SO.productname AS QT_productname,
        SO.productsku AS QT_productsku
    FROM (
        SELECT 
            SO.*, 
            RH.product_id AS QT_product_id,
            CASE 
                WHEN SO.productid = RH.product_id THEN 'SP Quà Tặng' 
                ELSE 'SP điều kiện' 
            END AS Type_SKU
        FROM og_reward.customer_reward_histories AS RH
        INNER JOIN SO ON RH.order_code = SO.ordernumber
        WHERE DATE(RH.created_at) >= '2025-01-01' 
            AND campaign_type IN (1,2) 
            AND RH.status = 1 
    ) AS A 
    LEFT JOIN SO ON A.QT_product_id = SO.productid AND A.ordernumber = SO.ordernumber
)

--Step 3: Retrieve final dataset for rewarded products with SKU 'MK%'
SELECT 
    order_date, 
    ordernumber, 
    productid, 
    productname, 
    productsku,
    Type_SKU, 
    QT_product_id, 
    QT_productname, 
    QT_productsku,
    SUM(quantity) AS total_quantity,
    SUM(subtotal) AS total_subtotal
FROM QT 
WHERE QT_productsku LIKE 'MK%'
GROUP BY 1,2,3,4,5,6,7,8,9
ORDER BY order_date DESC, ordernumber DESC, total_quantity DESC;
```
---

## 🔍 SQL Query Breakdown

### **Step 1: Extract Sales Orders (`SO` CTE)**
✔ Pulls **sales order data** from the `og_order` database.  
✔ Classifies orders by **region** (`Miền Bắc` or `Miền Nam`) based on the warehouse code.  
✔ Filters out **canceled orders** and restricts to a **specific warehouse (`CENHCM01`)**.  
✔ Aggregates **order subtotal, quantity, unique customers, and revenue**.  

---

### **Step 2: Identify Rewarded Products (`QT` CTE)**
✔ Retrieves **reward history** from the `og_reward` database.  
✔ Matches sales orders with **rewarded products** using `order_code`.  
✔ Categorizes products into:
   - 🏆 **"SP Quà Tặng" (Gift Product)**  
   - 📌 **"SP điều kiện" (Conditional Product)**  
✔ Ensures only **active rewards** from specific **campaign types (1,2)** are considered.  

---

### **Step 3: Final Report on Promotional Products**
✔ Filters rewarded products with **SKU starting with `MK%`**.  
✔ Groups data by **order date, product details, and gift type**.  
✔ Aggregates **total quantity of gifts and total revenue impact**.  
✔ Sorts results by **latest orders and highest gift quantity**.  

---

## 📈 Business Impact
**Use this data to optimize future promotions, keep track of inventories, boost customer engagement, and drive revenue!**
