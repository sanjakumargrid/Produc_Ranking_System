# Module: Getting Along with JPA (Entity Mapping & Relationships)

## 🎯 Task Goal
To map Object-Oriented Java models directly to Relational Database tables using the Java Persistence API (JPA) and Hibernate, focusing heavily on performance optimization and strict relationship mapping.

## 📖 Glossary of Key Terms Explored
* **JPA (Java Persistence API):** A specification in Java for accessing, persisting, and managing data. Hibernate is the actual underlying implementation of this specification.
* **`@Entity` & `@Id`:** Annotations that tell Hibernate, "This Java class represents a database table, and this specific field is the Primary Key."
* **`@OneToOne` & `@MapsId`:** Used when two entities are strictly bound together (like a Product and its specific Metrics). `@MapsId` tells the child entity to borrow the exact same Primary Key as the parent.
* **`FetchType.LAZY` vs `EAGER`:** Determines *when* related data is loaded. Eager loads everything immediately. Lazy loads it only when the getter method is explicitly called.
* **N+1 Query Problem:** A massive performance flaw where fetching a list of 100 products results in 1 initial query + 100 individual queries to fetch their reviews.
* **Cascading (`CascadeType.ALL`):** A setting that cascades entity state transitions. If I delete a Product, its Reviews should automatically be deleted.

## 🚀 Progress & Implementation Details

### 1. Highly Cohesive `@OneToOne` Mapping
* **What was done:** Linked `Product` and `ProductMetrics` using a `@OneToOne` association. Instead of giving `ProductMetrics` its own auto-generated ID, I used `@MapsId`.
* **Why it matters:** This ensures the `ProductMetrics` row shares the exact same Primary Key (ID) as the `Product` row. This optimizes database lookups; to find metrics for Product #5, Hibernate instantly looks for Metrics #5 without needing a complex `JOIN` on a foreign key column.

### 2. Preventing the N+1 Problem via Lazy Loading
* **What was done:** Configured the `Review` and `ProductOffer` entities to link back to the `Product` using `@ManyToOne(fetch = FetchType.LAZY)`.
* **Why it matters:** If we display a list of 50 top-ranked products, we don't need their thousands of reviews loaded into memory. `FetchType.LAZY` prevents Hibernate from generating massive, unnecessary SQL `JOIN` statements. The reviews are replaced with "Proxies" and only fetched if we explicitly call `product.getReviews()`, saving heap memory and database bandwidth.

### 3. Maintaining Data Integrity (Cascading)
* **What was done:** Applied `cascade = CascadeType.ALL` and `orphanRemoval = true` on the `Product` side of the `@OneToMany` relationships.
* **Why it matters:** This protects relational integrity. If a Product is discontinued and deleted from the database, the `orphanRemoval` flag guarantees that all associated reviews and offers are automatically purged from the database, preventing orphaned records and database bloat.
