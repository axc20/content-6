# Database Services

## RDS (Relational Database Service)

- Simplifies using databases by handling hardware, provisioning, setup, backups, patching, storage autoscaling, and more
- Offers many database engines including MySQL, Postgres, SQL Server, Oracle, and MariaDB
- Very similar to running a MySQL database on EC2

## Aurora

- Database engine option optimized for the cloud
- Compatible with both MySQL and Postgres
- Innovative shared storage architecture decouples storage from compute
- Serverless option

### Structure

- Table (primary keys, foreign keys)
- Relationships
- Schemas

### Storage

- Decoupled compute and storage
- Multi AZ replication

### Scale (more throughput)

- Vertical (better machine)
- Horizontal (more machines)
- Cluster endpoint (writing data)
- Reader endpoint

### Access

- Database connection + raw SQL
- Object relational mapper (ORM)
- Data API (REST endpoint)

## DynamoDB

- NoSQL database engine
- Optimized for key value lookups
- Great for high performance workloads with predictable performance
- Fully managed
- Pay for what you use model
- Supports transactions

### Structure

- Tables (partition key + sort key, GSI (global secondary index), attributes)
- Items

### Storage

- Hashing function: output is the location/partition where the record is located

### Scale

- Add more / split partitions

### Access

- AWS SDK
- Language specific ORM

## When to Use What

- [AWS Aurora VS DynamoDB](https://www.youtube.com/watch?v=crHwekf0gTA)
- Aurora
  - When you want to keep the door open for flexible access patterns
  - When you want to perform bulky, relational queries
  - When you want to enforce schema constraints
  - When you want to use a ubiquitous access language (SQL)
- DynamoDB
  - When you have predictable access patterns
  - When your search key is known in advance
  - When you are willing to sacrifice flexibility for ultra fast and consistent performance

### Mental Model and Usage/Problem Example

- Aurora: Start with a data relationship
  - Orders: OrderId, CustomerId, Date
  - Customers: CustomerId, Name, Address
- DynamoDB: Start with your access patterns
  - "I want to be able to access all OrderIds for a Customer by their CustomerId"
  - CustomerOrders: CustomerId (partition key), OrderId (sort key), CustomerDetails
