# Amazon-Transportation-data-analysis
## Project Overview

This project focuses on analyzing an Amazon-like order transportation and logistics system using SQL.
The objective is to evaluate order fulfillment performance, delivery efficiency, carrier reliability, shipping methods, and regional behavior through structured relational data.

The project is designed around operational analytics, not sales analytics, and reflects real-world logistics scenarios such as delayed deliveries, order cancellations, and carrier performance variation.

## Database Schema

The database consists of four relational tables:

### Customers
Stores customer details and region information.

### Orders
Acts as the central table that tracks the order lifecycle, shipping method, and order status.

### Transportation
Contains shipment-related details including carrier assignment, delivery date, and shipping cost.

### Carriers
Stores logistics carrier information along with their operational regions.

## Relationship Overview

One customer can place multiple orders

Each order has one transportation record

Each transportation record is handled by one carrier

This relational structure enables end-to-end tracking from order placement to delivery execution.
## Project Summary:

1. **Built and Analyzed a Dataset of 200+ Records:** Created and managed a relational database with over 200 rows of data across four tables (`Orders`, `Transportation`, `Carriers`, and `Customers`), using SQL to simulate Amazon's order transportation system.

2. **Delivered Key Insights Across 10+ Metrics:** Executed 15+ complex SQL queries to evaluate delivery timeliness, regional performance, carrier reliability, and shipping method efficiency, identifying a 15% delay rate and high cancellation rates in specific regions.

3. **Enhanced Logistics Optimization:** Pinpointed performance drops and bottlenecks in the transportation process, recommending improvements based on quantitative insights that could potentially reduce delays by up to 20%.

---

### **1. Order Status Distribution**
```sql
SELECT 
    Status, 
    COUNT(*) AS TotalOrders
FROM 
    Orders
GROUP BY 
    Status;
```
- **Purpose**: Understand the distribution of orders by status (e.g., Delivered, Processing, Cancelled).

---

### **2. Average Delivery Time by Carrier**
```sql
SELECT 
    t.CarrierID, 
    c.CarrierName, 
    AVG(DATEDIFF(t.DeliveryDate, o.OrderDate)) AS AvgDeliveryTime
FROM 
    Transportation t
JOIN 
    Orders o ON t.OrderID = o.OrderID
JOIN 
    Carriers c ON t.CarrierID = c.CarrierID
WHERE 
    t.DeliveryDate IS NOT NULL
GROUP BY 
    t.CarrierID, c.CarrierName;
```
- **Purpose**: Measure the performance of carriers based on average delivery times.

---

### **3. Orders Delivered Late**
```sql
SELECT 
    o.OrderID, 
    o.OrderDate, 
    t.DeliveryDate, 
    DATEDIFF(t.DeliveryDate, o.OrderDate) AS DeliveryTime
FROM 
    Orders o
JOIN 
    Transportation t ON o.OrderID = t.OrderID
WHERE 
    t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5;
```
- **Purpose**: Identify orders delivered after 5 days, indicating potential delays.

---

### **4. Regional Order Distribution**
```sql
SELECT 
    c.Region, 
    COUNT(*) AS TotalOrders
FROM 
    Customers c
JOIN 
    Orders o ON c.CustomerID = o.CustomerID
GROUP BY 
    c.Region
ORDER BY 
    TotalOrders DESC;
```
- **Purpose**: Track the number of orders in each region and identify high-demand areas.

---

### **5. Cancelled Orders by Region**
```sql
SELECT 
    c.Region, 
    COUNT(*) AS CancelledOrders
FROM 
    Customers c
JOIN 
    Orders o ON c.CustomerID = o.CustomerID
WHERE 
    o.Status = 'Cancelled'
GROUP BY 
    c.Region
ORDER BY 
    CancelledOrders DESC;
```
- **Purpose**: Identify regions with the highest number of canceled orders.

---

### **6. Popular Shipping Methods**
```sql
SELECT 
    ShippingMethod, 
    COUNT(*) AS TotalOrders
FROM 
    Orders
GROUP BY 
    ShippingMethod
ORDER BY 
    TotalOrders DESC;
```
- **Purpose**: Determine which shipping methods are most frequently chosen by customers.

---

### **7. Cost Efficiency of Carriers**
```sql
SELECT 
    t.CarrierID, 
    c.CarrierName, 
    AVG(t.ShippingCost) AS AvgShippingCost
FROM 
    Transportation t
JOIN 
    Carriers c ON t.CarrierID = c.CarrierID
GROUP BY 
    t.CarrierID, c.CarrierName
ORDER BY 
    AvgShippingCost ASC;
```
- **Purpose**: Evaluate carriers based on their average shipping costs.

---

### **8. High-Performing Carriers**
```sql
SELECT 
    t.CarrierID, 
    c.CarrierName, 
    COUNT(o.OrderID) AS DeliveredOrders
FROM 
    Transportation t
JOIN 
    Orders o ON t.OrderID = o.OrderID
JOIN 
    Carriers c ON t.CarrierID = c.CarrierID
WHERE 
    o.Status = 'Delivered'
GROUP BY 
    t.CarrierID, c.CarrierName
ORDER BY 
    DeliveredOrders DESC;
```
- **Purpose**: Identify carriers with the highest number of successfully delivered orders.

---

### **9. Delays by Shipping Method**
```sql
SELECT 
    o.ShippingMethod, 
    COUNT(CASE WHEN DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders
FROM 
    Orders o
JOIN 
    Transportation t ON o.OrderID = t.OrderID
WHERE 
    t.DeliveryDate IS NOT NULL
GROUP BY 
    o.ShippingMethod;
```
- **Purpose**: Track delays based on the shipping method used.

---

### **10. Top Customers by Orders**
```sql
SELECT 
    c.CustomerID, 
    c.Name, 
    COUNT(o.OrderID) AS TotalOrders
FROM 
    Customers c
JOIN 
    Orders o ON c.CustomerID = o.CustomerID
GROUP BY 
    c.CustomerID, c.Name
ORDER BY 
    TotalOrders DESC
LIMIT 10;
```
- **Purpose**: Identify the top 10 customers with the most orders.

---

### **11. Total Shipping Cost by Region**
```sql
SELECT 
    c.Region, 
    SUM(t.ShippingCost) AS TotalShippingCost
FROM 
    Customers c
JOIN 
    Orders o ON c.CustomerID = o.CustomerID
JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    c.Region
ORDER BY 
    TotalShippingCost DESC;
```
- **Purpose**: Evaluate regions with the highest shipping costs.

---

### **12. Daily Order Trends**
```sql
SELECT 
    DATE(o.OrderDate) AS OrderDate, 
    COUNT(*) AS TotalOrders
FROM 
    Orders o
GROUP BY 
    DATE(o.OrderDate)
ORDER BY 
    OrderDate ASC;
```
- **Purpose**: Track the trend of orders over time and identify peak days.

---


### Tracking the performance drop of orders
**involves analyzing trends and identifying areas where logistics or operations may have faltered.**
**We’ll focus on  canceled orders, delayed deliveries, and changes in overall order statuses over time. Here’s how we can analyze this:**

---

### **1. Trend in Order Status Over Time**
This query tracks the number of canceled, delayed, and delivered orders on a daily basis.

```sql
SELECT 
    DATE(o.OrderDate) AS OrderDate, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
    COUNT(CASE WHEN o.Status = 'Delivered' THEN 1 END) AS DeliveredOrders
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    DATE(o.OrderDate)
ORDER BY 
    OrderDate ASC;
```
- **Insight**: This query reveals if the number of canceled or delayed orders is increasing over time, which may point to performance drops.

---

### **2. Regional Drop in Performance**
Identify regions with a worsening trend in delayed and canceled orders.

```sql
SELECT 
    c.Region, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
    COUNT(CASE WHEN o.Status = 'Delivered' THEN 1 END) AS DeliveredOrders
FROM 
    Customers c
JOIN 
    Orders o ON c.CustomerID = o.CustomerID
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    c.Region
ORDER BY 
    CancelledOrders DESC, DelayedOrders DESC;
```
- **Insight**: Pinpoints regions with declining delivery performance or high cancellations.

---

### **3. Carrier Drop in Performance**
Evaluate carriers that show a significant increase in delayed deliveries or cancellations.

```sql
SELECT 
    t.CarrierID, 
    c.CarrierName,
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders
FROM 
    Transportation t
JOIN 
    Orders o ON t.OrderID = o.OrderID
JOIN 
    Carriers c ON t.CarrierID = c.CarrierID
GROUP BY 
    t.CarrierID, c.CarrierName
ORDER BY 
    CancelledOrders DESC, DelayedOrders DESC;
```
- **Insight**: Identifies carriers contributing to performance drops and allows you to focus improvement efforts.

---

### **4. Shipping Method Performance**
Determine which shipping methods are leading to higher delays or cancellations.

```sql
SELECT 
    o.ShippingMethod, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
    COUNT(CASE WHEN o.Status = 'Delivered' THEN 1 END) AS DeliveredOrders
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    o.ShippingMethod
ORDER BY 
    CancelledOrders DESC, DelayedOrders DESC;
```
- **Insight**: Pinpoints shipping methods that may be leading to order delays or cancellations.

---

### **5. Overall Performance Drop**
Analyze the percentage of delayed or canceled orders relative to total orders.

```sql
SELECT 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) * 100.0 / COUNT(*) AS CancelledPercentage,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) * 100.0 / COUNT(*) AS DelayedPercentage
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID;
```
- **Insight**: Provides overall metrics to assess the extent of performance drops.

---

