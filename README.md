SQL Analysis Questions and Explanations

This section documents the analytical questions addressed in the project and explains how each SQL query contributes to understanding order transportation performance.

Data Validation and Initial Exploration
SELECT * FROM Orders;
SELECT * FROM Transportation;
SELECT * FROM Carriers;
SELECT * FROM Customers;


Purpose
These queries are used to verify data availability, structure, and consistency across all tables before performing analysis. This step ensures that joins and aggregations operate on complete and valid datasets.

1. Order Status Distribution
SELECT 
    Status, 
    COUNT(*) AS TotalOrders
FROM 
    Orders
GROUP BY 
    Status;


Analysis Objective
To understand how orders are distributed across different lifecycle stages such as Delivered, Processing, Shipped, and Cancelled.

Why this works
Grouping by Status provides a high-level operational snapshot. This forms the baseline for identifying cancellation volume and delivery success rates.

2. Average Delivery Time by Carrier
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


Analysis Objective
To evaluate carrier performance based on average delivery duration.

Why this logic is correct

DATEDIFF measures actual delivery time from order placement to delivery.

Orders without delivery dates are excluded to avoid skewed averages.

Grouping by carrier isolates individual carrier performance.

3. Orders Delivered Late
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
    t.DeliveryDate IS NOT NULL 
    AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5;


Analysis Objective
To identify delayed orders using a delivery threshold of more than five days.

Business Interpretation
Orders exceeding this threshold indicate logistics inefficiencies or carrier delays.

4. Regional Order Distribution
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


Analysis Objective
To identify regions with the highest order volume.

Why this matters
High-demand regions require stronger logistics support and optimized carrier allocation.

5. Cancelled Orders by Region
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


Analysis Objective
To detect regions with higher cancellation frequency.

Business Interpretation
High cancellations may indicate delivery delays, carrier issues, or customer dissatisfaction in those regions.

6. Popular Shipping Methods
SELECT 
    ShippingMethod, 
    COUNT(*) AS TotalOrders
FROM 
    Orders
GROUP BY 
    ShippingMethod
ORDER BY 
    TotalOrders DESC;


Analysis Objective
To understand customer preference for shipping methods.

Why this matters
Helps assess demand for Standard, Expedited, and Same-day delivery options.

7. Cost Efficiency of Carriers
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


Analysis Objective
To evaluate carriers based on average shipping cost.

Business Interpretation
Identifies cost-efficient carriers and supports cost-performance trade-off decisions.

8. High-Performing Carriers
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


Analysis Objective
To identify carriers with the highest successful delivery count.

Why this matters
Highlights reliable carriers contributing most to order completion.

9. Delays by Shipping Method
SELECT 
    o.ShippingMethod, 
    COUNT(CASE 
        WHEN DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 
    END) AS DelayedOrders
FROM 
    Orders o
JOIN 
    Transportation t ON o.OrderID = t.OrderID
WHERE 
    t.DeliveryDate IS NOT NULL
GROUP BY 
    o.ShippingMethod;


Analysis Objective
To evaluate which shipping methods are more prone to delays.

Business Interpretation
Helps assess whether premium shipping methods consistently meet delivery expectations.

10. Top Customers by Order Volume
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


Analysis Objective
To identify high-frequency customers.

Why this matters
Supports customer segmentation and prioritization strategies.

11. Total Shipping Cost by Region
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


Analysis Objective
To evaluate regional contribution to overall shipping expenses.

12. Daily Order Trends
SELECT 
    DATE(o.OrderDate) AS OrderDate, 
    COUNT(*) AS TotalOrders
FROM 
    Orders o
GROUP BY 
    DATE(o.OrderDate)
ORDER BY 
    OrderDate ASC;


Analysis Objective
To track order volume trends over time and identify peak periods.

Performance Drop Analysis

This section focuses on identifying operational degradation through trends in cancellations and delivery delays.

1. Trend in Order Status Over Time
SELECT 
    DATE(o.OrderDate) AS OrderDate, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL 
        AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
    COUNT(CASE WHEN o.Status = 'Delivered' THEN 1 END) AS DeliveredOrders
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    DATE(o.OrderDate)
ORDER BY 
    OrderDate ASC;


Insight
Tracks operational stability over time and highlights periods of performance decline.

2. Regional Drop in Performance
SELECT 
    c.Region, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL 
        AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
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


Insight
Identifies regions with declining logistics performance.

3. Carrier Drop in Performance
SELECT 
    t.CarrierID, 
    c.CarrierName,
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL 
        AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders
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


Insight
Highlights carriers contributing to performance degradation.

4. Shipping Method Performance
SELECT 
    o.ShippingMethod, 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) AS CancelledOrders,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL 
        AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) AS DelayedOrders,
    COUNT(CASE WHEN o.Status = 'Delivered' THEN 1 END) AS DeliveredOrders
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID
GROUP BY 
    o.ShippingMethod
ORDER BY 
    CancelledOrders DESC, DelayedOrders DESC;


Insight
Compares shipping methods across cancellations, delays, and successful deliveries.

5. Overall Performance Metrics
SELECT 
    COUNT(CASE WHEN o.Status = 'Cancelled' THEN 1 END) * 100.0 / COUNT(*) AS CancelledPercentage,
    COUNT(CASE WHEN t.DeliveryDate IS NOT NULL 
        AND DATEDIFF(t.DeliveryDate, o.OrderDate) > 5 THEN 1 END) * 100.0 / COUNT(*) AS DelayedPercentage
FROM 
    Orders o
LEFT JOIN 
    Transportation t ON o.OrderID = t.OrderID;


Insight
Provides normalized performance indicators to assess overall operational health.
