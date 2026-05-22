# Module: Setting Up (Environment & Database Infrastructure)

## 🎯 Task Goal
To establish a robust, scalable persistence layer by configuring a relational database, integrating it with the Spring Boot application, and optimizing the connection pipeline for high performance.

## 📖 Glossary of Key Terms Explored
* **PostgreSQL:** An advanced, enterprise-class open-source relational database. Used as the primary data store for production and development.
* **Docker Compose:** A tool for defining and running multi-container Docker applications. It guarantees that every developer working on the project has the exact same database environment.
* **HikariCP:** A "zero-overhead" production-ready JDBC connection pool. It is the default connection pool in Spring Boot.
* **Connection Pooling:** Instead of opening and closing a database connection for every single query (which is extremely slow), a pool keeps a set of connections open and reuses them.
* **DDL (Data Definition Language) Auto:** A Hibernate feature (`spring.jpa.hibernate.ddl-auto`) that automatically generates SQL commands to create or update database tables based on our Java classes.

## 🚀 Progress & Implementation Details

### 1. Reproducible Infrastructure via Docker
* **What was done:** Configured a `docker-compose.yml` file to spin up a PostgreSQL instance mapped to port `5432`.
* **Why it matters:** This containerizes the database layer. It prevents the "it works on my machine" problem by explicitly defining the exact version of Postgres, the required port, and the default credentials in an isolated environment.

### 2. Spring Boot Persistence Configuration
* **What was done:** Updated `application-dev.properties` to securely connect to the PostgreSQL database using the `org.postgresql.Driver`.
* **Why it matters:** By defining `spring.datasource.url`, `username`, and `password`, we instruct the Spring `DataSource` bean on how to route traffic to the container.

### 3. Schema Synchronization (`ddl-auto`)
* **What was done:** Set `spring.jpa.hibernate.ddl-auto=update` in development.
* **Why it matters:** As the E-Commerce domain model evolves (e.g., adding a new field to `Product`), Hibernate automatically calculates the difference between the Java `@Entity` and the actual Postgres table, dynamically issuing an `ALTER TABLE` command to keep them perfectly in sync without manual SQL scripts.

### 4. Connection Pool Optimization
* **What was done:** Relied on Spring Boot’s default HikariCP configuration, verifying its initialization in the application startup logs.
* **Why it matters:** Because our application has a background `@Scheduled` task that constantly queries the database to calculate ranking scores, establishing a new connection every hour would throttle the system. HikariCP holds a pool of active connections, drastically reducing query latency and preventing connection timeouts under high load.
