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

## Designing TikTok

Functional Requirements for TikTok System Design
- User Profile
- Uploading and streaming short video clips.
- Creating and sharing video content.
- Following user-profiles and exploring curated video feeds.
- Liking, disliking, and commenting on videos.
- Discovering new videos based on personalized recommendations.

Assumptions:
- MAU: 1 billion monthly active users.
- 50% of users use TikTok daily.
- 10% of users post videos

Estimations:
Query per second (QPS) estimate:
- Daily active users (DAU) = 1 billion * 50% = 500 million
- Posts QPS = 500 million / 24 hour / 3600 seconds = ~5787 posts/sec
- Peek QPS = 2 * QPS = ~11574

We will only estimate media storage here.
 - Average user profile size: 1MB
 - Total User Profiles Storage: 1 billion * 1 MB = 1 Petabyte (PB)
 - Average Video Metadata: 500 KB
 - Daily Video Metadata Storage: 50 million * 500 KB = 25 Terabytes (TB)
 - Monthly Video Metadata Storage: 25 TB * 30 days = 750 TB
 - Average Video size: 20MB.
 - Total Video Streams Storage: 50 million * 20 MB = 1 Petabyte (PB) daily
 - Monthly Video Streams Storage: 1 PB * 30 days = 30 PB

![Alt text](tiktok.png?raw=true "Application architecture")

Considerations:
- CDN Solutions for Video could help, like AWS CloudFront, Cloudflare Stream
- For Recommendation System we need real-time analytics, typically:
  - Event-driven architecture (Kafka, Kinesis) to process user interactions.
  - Machine Learning Models (e.g., TensorFlow, PyTorch) to suggest videos based on user behavior.
  - Graph Databases (Neo4j, Amazon Neptune) for social connections and trends.
- Relational DBs (PostgreSQL) are utilized for storing structured data, such as user profiles, relationships, and video metadata.
- NoSQL databases, like Redis and Cassandra, are utilized for handling unstructured or semi-structured data, such as user interactions (likes, comments, shares) and scalable storage of videos.
- Cloud-based object storage solutions like Amazon S3 or Google Cloud Storage are employed for storing video content in its original form.
- Fanout Services are responsible for efficiently distributing uploaded videos to users' feeds, ensuring a personalized experience.
- The Cache services play a crucial role in optimizing content delivery by storing personalized feeds, metadata, and trending content. Utilizing Redis as an in-memory data store, it caches frequently accessed data, reducing latency for users accessing personalized feeds and trending videos.

## Design online movie ticketing system BookMyShow

Functional Requirements
- City Selection: The portal should provide a list of cities where theatres are available, allowing users to select their preferred location.
- Movie Listings: Based on the selected city, the portal should display all movies currently running in that city.
- Cinema and Show Selection: For a selected movie, the portal should list cinemas running the movie, along with available showtimes.
- Ticket Booking: Users should be able to select a show at a specific theatre and proceed to book tickets seamlessly.
- Ticket Notifications: After booking, the system should send a copy of the tickets to the user via SMS or email for confirmation and record.
- Seating Arrangement Display: The portal should visually display the seating arrangement of the selected cinema hall.
- Seat Selection: Users should have the ability to choose multiple seats from the seating arrangement as per their preference.
- Ticket Serving Order: The system should ensure tickets are allocated on a First In, First Out (FIFO) basis to handle multiple concurrent bookings fairly and accurately.

![Alt text](bookmyshow.png?raw=true "Application architecture")

Considerations:
- What will happen if multiple users will try to book the same ticket using different platforms? How to solve this problem? 
  - The theatre’s server needs to follow a timeout locking mechanism strategy where a seat will be locked temporarily for a user for a specific time session (for example, 5-10 minutes). If the user is not able to book the seat within that timeframe then release the seat for another user. This should be done on a first come first serve basis.
- Redis Cluster: We have a set of Redis clusters each having their own set of nodes(or replicas), configured on the basis of use cases. It is primarily being used for app server caching, storing movie metadata(description, genre, languages, actors’ info, release date, film board certification number, user reviews and comments, etc) and for the seat-blocking mechanism, during tentative booking(prior to payment)
- Since the booking process is quintessentially a transactional one, we need to enforce strict ACID compliance. Also we need to store many other relational information about operators, theatres, halls, seats, etc for our local TMS. To achieve both the goals, we have used MySQL in this exercise(but PostgreSQL, Oracle, etc can also be used). We can configure partitions based on regions, if the user base grows. Replication can be similar to redis.
- Messaging Queue: In order to optimise the booking process for fast response time and efficient handling of requests, we should adopt an asynchronous approach for sending notifications. This can be achieved by utilising a message-queue system based on Kafka. The notification service will generate messages and publish them to the queue, while consumers of the queue will be responsible for delivering emails, SMS, and push
notifications to the intended recipients.

## Design Instagram

Functional Requirements for Instagram System Design:

- Post photos and videos: The users can post photos and videos on Instagram.
- Follow and unfollow users: The users can follow and unfollow other users on Instagram.
- Like or dislike posts: The users can like or dislike posts of the accounts they follow.
- Search photos and videos: The users can search photos and videos based on captions and location.
- Generate news feed: The users can view the news feed consisting of the photos and videos (in chronological order) from all the users they follow.

Estimations:
Query per second (QPS) estimate:
- Daily active users (DAU) = 1 billion * 50% = 500 million
- On average, each user sends 20 requests (of any type) per day to our service
- QPS = 500 million * 20 / 24 hour / 3600 seconds = ~115 740 q/s
- Peek QPS = ~234 481 q/s
- Assume 10% of DAU make 2 posts a day
- Posts QPS = 500 million * 0.1 * 2 / 24 hour / 3600 seconds = ~1157 posts/sec
- Peek posts QPS = 2 * QPS = ~2314
- We can consider 3 MB as the maximum size of each photo and 150 MB as the maximum size of each video uploaded on Instagram
- Assume 60 million photos and 35 million videos are shared on Instagram per day
- Storage: 60 million * 3 MB + 35 million * 150 MB = 180 million MB + 5250 MB = 5430 million MG = ~5.43 PB/day
- Storage per year = 5.43 * 365 = 1981.95 PB
- AWS S3, Standard storage: ~$0.023 per GB per month, Cost per month: $46,643,825, Annual cost: ~$559.7 million
- Microsoft Azure Blob Storage, Hot tier: ~$0.018 per GB per month, Cost per month: $36,503,863, Annual cost: ~$438 million
- Bandwidth: = ~5.43 PB/day / 24 / 3600 ~= 62.84 GB/s ~= 502.8 Gbps
- Incoming bandwidth ~= 502.8 Gbps
- Let’s say the ratio of readers to writers is 100:1.
- Required outgoing bandwidth ~= 100 * 502.8 Gbps ~= 50.28 Tbps

![Alt text](instagram.png?raw=true "Application architecture")

User Service:
- Handles user registration, login, authentication, and profile management.
- Stores user data like username, email, bio, profile picture, etc.
- Integrates with social authentication providers (e.g., Facebook, Google).

Post Service:
- Handles photo and video uploads, editing, and deletion.
- Stores post metadata like caption, hashtags, location, timestamp, etc.Processes uploaded media for resizing, filtering, and thumbnail generation.
- Manages photo and video transcoding for different devices and resolutions.

Feed Service:
- Generates personalized news feeds for each user based on their follows, likes, activity, and engagement.
- Leverages a distributed system like Apache Kafka or RabbitMQ for real-time updates and notifications.
- Utilizes a cache layer like Redis for fast feed retrieval and reduced database load.

Storage Service:
- Stores uploaded photos and videos efficiently and reliably.
- Utilizes a scalable object storage solution like Amazon S3, Google Cloud Storage, or Azure Blob Storage.
- Implements redundancy and disaster recovery mechanisms for data protection.

Search Service:
- Enables searching for users, hashtags, and locations.
- Indexes users, posts, and hashtags based on relevant parameters.
- Employs efficient indexing and search algorithms for fast and accurate results.

Notification Service:
- Informs users about relevant events like likes, comments, mentions, and follows.
- Pushes notifications to mobile devices through platforms like Firebase Cloud Messaging or Amazon SNS.
- Leverages a queueing system for asynchronous notification delivery.

Analytics Service:
- Tracks user engagement, post performance, and overall platform usage.
- Gathers data on views, likes, comments, shares, and clicks.
- Provides insights to improve user experience, optimize content recommendations, and target advertising.

## Design Youtube

Functional requirements:
- Ability to upload videos fast
- Smooth video streaming
- Ability to change video quality
- Low infrastructure cost
- High availability, scalability, and reliability requirements
- Clients supported: mobile apps, web browser, and smart TV

Assumptions:
- Assume the product has 5 million daily active users (DAU).
- Users watch 5 videos per day.
- 10% of users upload 1 video per day.
- Assume the average video size is 300 MB.
- Total daily storage space needed: 5 million * 10% * 300 MB = 150TB
- CDN cost.
  - When cloud CDN serves a video, you are charged for data transferred out of the
CDN.
  - Let us use Amazon’s CDN CloudFront for cost estimation (Figure 14-2) [3]. Assume
100% of traffic is served from the United States. The average cost per GB is $0.02.
For simplicity, we only calculate the cost of video streaming.
  - 5 million * 5 videos * 0.3GB * $0.02 = $150,000 per day.

![Alt text](youtube.png?raw=true "Application architecture")

- User: A user watches YouTube on devices such as a computer, mobile phone, or smart
TV.
- Load balancer: A load balancer evenly distributes requests among API servers.
- API servers: All user requests go through API servers except video streaming.
- Metadata DB: Video metadata are stored in Metadata DB. It is sharded and replicated to
meet performance and high availability requirements.
- Metadata cache: For better performance, video metadata and user objects are cached.
- Original storage: A blob storage system is used to store original videos. A quotation in
Wikipedia regarding blob storage shows that: “A Binary Large Object (BLOB) is a
collection of binary data stored as a single entity in a database management system” [6].
- Transcoding servers: Video transcoding is also called video encoding. It is the process of
converting a video format to other formats (MPEG, HLS, etc), which provide the best
video streams possible for different devices and bandwidth capabilities.
• Transcoded storage: It is a blob storage that stores transcoded video files.
• CDN: Videos are cached in CDN. When you click the play button, a video is streamed
from the CDN.
• Completion queue: It is a message queue that stores information about video transcoding
completion events.
• Completion handler: This consists of a list of workers that pull event data from the
completion queue and update metadata cache and database.

## References
[Online Movie Ticket Booking Platform - System Design (e.g. BookMyShow)](https://medium.com/@prithwish.samanta/online-movie-ticket-booking-platform-system-design-e-g-bookmyshow-69048440901c)

[Alex Xu, System Design Interview – An insider's guide, 2020]