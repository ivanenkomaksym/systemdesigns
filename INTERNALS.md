# Pagination, filtering, and sorting

1. OData
```
GET /api/Products?$top=10&$skip=20&$filter=Category eq 'Electronics'&$orderby=Price desc
```

2. GraphQL
```
query {
  products(
    first: 10
    after: "cursor123"
    filter: { category: "Electronics" }
    sort: { field: "price", order: DESC }
  ) {
    edges {
      node {
        id
        name
        category
        price
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

| **Feature**	| **OData** |	**GraphQL** |
|-------------------|-------------------------|-------------------------------|
| **Pagination Type**	| 	Offset-based ($top, $skip)	|	Cursor-based (first, after)|
| **Filtering**	|	$filter=...	|	filter: { ... }|
| **Sorting**	|	$orderby=...	|	sort: { field: ..., order: ... }|
| **Flexibility**	|	Predefined schema**	|	Customizable queries|
| **Over-fetching**	|	May return extra data**	|	Fetch only required fields|

**SQL vs NoSQL: Justification Based on Use Case, Read-Heavy Workloads, and Scalability**  

When choosing between **SQL (Relational Databases)** and **NoSQL (Non-Relational Databases)**, consider the **use case**, **read-heavy nature**, and **scalability** of your application.

---

## **🔹 1. Use Case Justification**  

| **Factor**        | **SQL (Relational DBs)** | **NoSQL (Non-Relational DBs)** |
|-------------------|-------------------------|-------------------------------|
| **Data Structure** | Well-structured, predefined schema (tables, rows, relationships). | Flexible, schema-less (documents, key-value, graphs, column-family). |
| **Use Case** | Ideal for transactional systems (e.g., Banking, ERP, CRM). | Suitable for dynamic or semi-structured data (e.g., IoT, social media, real-time analytics). |
| **ACID Compliance** | Strong consistency (Atomicity, Consistency, Isolation, Durability). | Eventual consistency (CAP Theorem: choose between Consistency, Availability, and Partition tolerance). |
| **Joins & Relationships** | Efficient support for complex joins & normalized data. | Not designed for joins; optimized for fast access patterns. |

### **💡 Example Use Cases**  
✅ **SQL** → Banking, E-commerce transactions, Inventory Management.  
✅ **NoSQL** → Social Media Feeds, IoT Data Storage, Caching Systems.  

---

## **🔹 2. Read-Heavy Nature**  

### **✅ When to Choose SQL for Read-Heavy Workloads**  
- 🔹 If **read queries are complex** (e.g., analytics, aggregations, reporting).  
- 🔹 When **data consistency is critical** (e.g., financial transactions, inventory management).  
- 🔹 **Indexing & Query Optimization** in SQL databases (e.g., PostgreSQL, MySQL) improve read performance.  

**Example SQL Query:**  
```sql
SELECT CustomerID, SUM(OrderTotal)
FROM Orders
WHERE OrderDate >= '2024-01-01'
GROUP BY CustomerID
ORDER BY SUM(OrderTotal) DESC
LIMIT 10;
```

| **Use Case**        |	SQL |	NoSQL |
|-------------------|-------------------------|-------------------------------|
| Transactional Systems	|✅ Best (ACID compliance)	|❌ Not ideal|
| Analytics & Reporting	|✅ Optimized	|❌ Not ideal|
| Social Media Feeds	|❌ Slow for massive reads	|✅ Fast (Denormalized)|
| IoT & Real-time Data	|❌ Struggles at scale	|✅ Optimized for time-series data|
| E-commerce & Inventory	|✅ Strong consistency	|⚠️ Can work but requires additional constraints|