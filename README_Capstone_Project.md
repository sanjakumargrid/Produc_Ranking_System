# Module: Capstone Project (Business Logic & Transactions)

## 🎯 Task Goal
To integrate the JPA persistence layer seamlessly into the business logic (Service Layer), utilizing advanced querying, transaction management, and safe data retrieval patterns.

## 📖 Glossary of Key Terms Explored
* **Spring Data JPA (`JpaRepository`):** An abstraction layer that automatically generates standard CRUD (Create, Read, Update, Delete) SQL queries for us at runtime based on the interface definition.
* **Derived Query Methods:** Spring Data's ability to translate method names directly into SQL queries (e.g., `findByRankingScoreDesc()`).
* **ACID Properties:** Atomicity, Consistency, Isolation, Durability. The core principles that guarantee database transactions are processed reliably.
* **`@Transactional`:** A Spring annotation that wraps a method in an ACID-compliant database transaction.
* **Pagination (`Pageable`):** The process of breaking a massive dataset into smaller, manageable "pages" (chunks) to prevent memory overload.

## 🚀 Progress & Implementation Details

### 1. Intelligent Data Retrieval (Derived Queries)
* **What was done:** Created the `ProductMetricsRepository` extending `JpaRepository` and added the method `findTop10ByOrderByRankingScoreDesc()`.
* **Why it matters:** Instead of manually writing error-prone SQL strings like `SELECT * FROM product_metrics ORDER BY ranking_score DESC LIMIT 10`, Spring Data JPA parses the method name and safely constructs the highly-optimized native query under the hood.

### 2. ACID Transaction Management
* **What was done:** Applied the `@Transactional` annotation to the `updateRankingScores()` scheduled task and other mutation methods. Applied `@Transactional(readOnly = true)` to fetch methods like `getTopRankedProducts()`.
* **Why it matters:** * **Atomicity:** When iterating through thousands of products to update their ranking scores, if the server crashes halfway through, `@Transactional` forces the database to immediately rollback the entire batch. This prevents corrupted, half-updated dashboard data.
    * **Performance (Read-Only):** Flagging a transaction as `readOnly = true` tells Hibernate to completely disable "Dirty Checking" (tracking objects for modifications). This saves significant CPU and memory when we only intend to display data, not change it.

### 3. Scalable Fetching via Pagination
* **What was done:** Utilized `PageRequest.of(0, 10)` inside the `OfferService` to fetch the top 10 best offers.
* **Why it matters:** As the application scales to millions of offers, running `findAll()` would pull the entire database into the JVM, instantly causing an `OutOfMemoryError`. By passing a `Pageable` object to the repository, Hibernate automatically appends standard SQL `LIMIT` and `OFFSET` clauses, ensuring we only retrieve exactly what the frontend requires.
