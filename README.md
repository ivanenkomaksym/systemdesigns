# System designs

## Table of contents

## Design a Scalable REST API

Design and implement a RESTful API for a product catalog in an e-commerce system. The API should support CRUD operations for products and allow filtering by category and price range:
- Proper API design with REST principles.
- Handling pagination, filtering, and sorting.
- Use of DTOs and validation.
- Consideration of caching and rate-limiting.
- How you structure dependency injection and logging.

![Alt text](ecommerce.png?raw=true "Application architecture")

### Pagination, filtering, and sorting

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

## Designing Twitter

Assumptions:
- 300 million monthly active users.
- 50% of users use Twitter daily.
- Users post 2 tweets per day on average.
- 10% of tweets contain media.
- Data is stored for 5 years.

Estimations:
Query per second (QPS) estimate:
- Daily active users (DAU) = 300 million * 50% = 150 million
- Tweets QPS = 150 million * 2 tweets / 24 hour / 3600 seconds = ~3500
- Peek QPS = 2 * QPS = ~7000

We will only estimate media storage here.
- Average tweet size:
- tweet_id 64 bytes
- text 140 bytes
- media 1 MB
- Media storage: 150 million * 2 * 10% * 1 MB = 30 TB per day
- 5-year media storage: 30 TB * 365 * 5 = ~55 PB

![Alt text](twitter.png?raw=true "Application architecture")