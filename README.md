# 📊 TRACKING PROMOTIONAL GIFTS AND REVENUE Tracking Promotional Gifts and Revenue Impact

## 📌 Overview
This code tracks **promotional gifts** in terms of **cost, revenue uplift, and gift count**.  
The relevant data is spread across **multiple different databases** (mainly from`og_order` and `og_reward`),  
which need to be linked before extracting the final results.

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
