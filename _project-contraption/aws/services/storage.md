# Storage Services

## S3 (Simple Storage Service)

- General object storage on the cloud
- Useful in a variety of contexts
  - Website hosting
  - Database backups
  - Data processing pipelines
- Core concepts
  - Buckets: container of objects you want to store within a certain namespace
  - Objects: content you are storing in your buckets, can be many different file types
  - Retrieve by URL: `http://s3.amazonaws.com/{BUCKET_NAME}/{OBJECT_NAME}`
  - Retrieve programatically
- Storage classes
  - Standard (default; excellent availability, low latency, high throughput), Intelligent, Infrequent Access, Glacier
  - Each tier has different pricing, latency, availability
  - To reduce costs, you can use a dynamic model to shift data into different storage classes over time: Standard Tier (hot data) --> Infrequent Access --> Glacier (cold data)
  - Lifecycle Rules automate the data movement process
- Security
  - Public access is blocked by default
  - Data protection - highly durable and availability guarantees, encrpytion in transit and at rest
  - Access - access and resourced based controls with IAM
  - Auditing - access logs, action based logs, alarms
  - Intrastructure security - built on top of AWS cloud infrastructure
- Pricing
  - Dependent on storage class
  - Three main factors: storage, access (GET, POST, etc.), transfer

### Example use cases

1. Data Ingestion Pipeline
   - data --> Kinesis Data Firehose --> S3 --> Lambda
   - S3 events: You can get notified whenever objects are created, modified, deleted, or replicated.
   - Whenever a batch of data gets delivered to S3, we can trigger an event to invoke a lambda. Inside that event will be the bucket name and object key that was created. Then, lambda can perform some data processing, data aggregation, filtering, and then put that data back into S3 or store in a database for indexing purposes.
2. Analytics and Dashboarding
   - customer --SQL--> Athena --> S3 --> QuickSight
   - Athena allows you to perform SQL-style queries on all the content within your bucket provided that all that content conforms to a very similar schema
3. Event Driven Architectures
   - customer --raw image --> S3 --process image (put event)--> Lambda --notify--> AppSync --processed image is ready--> customer

### Links

- [How to use SQL to query S3 files with AWS Athena](https://www.youtube.com/watch?v=M5ptG0YaqAs)
- [AWS S3 Tutorial For Beginners](https://www.youtube.com/watch?v=tfU0JEZjcsg)
