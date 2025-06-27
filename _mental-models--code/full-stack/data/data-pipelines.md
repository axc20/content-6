# Data Pipelines / "Orchestration" / ETL (Extract, Transform, Load)

Data pipelines will put you in the top 1% of the market ([@svpino](https://x.com/svpino)). **Learn to move and process data at scale.**

1. Getting the data from its source
2. Processing and cleaning that data
3. Delivering the cleaned data to the right place

The challenge is designing resilient, scalable, and fault-tolerant systems.

A production-ready orchestration platform ([kestra.io](https://kestra.io/)) will need an event-driven architecture. Event-driven means you can kick off a workflow automatically based on different triggers. For instance, when somebody uploads a new file to a folder, an app updates a database table, or there's aa new message in a queue.

To build your first pipeline, you need to understand a couple of concepts:

1. Data Nodes: They represent any data you want to load, manipulate, or save.
2. Tasks: These are functions that will interact with the data.

A pipeline combines data nodes with tasks. It's that simple. Try a simple example with [taipy](https://taipy.io/).
