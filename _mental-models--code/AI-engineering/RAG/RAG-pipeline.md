# RAG Pipeline

1. @llama_index parser
2. @firecrawl_dev for web pages
3. Process CSVs and tables to a SQL table, and use text-to-sql as a tool for your LLM
4. Use a multimodal model to describe images and embed the description
5. Query expansion: Generate multiple queries from the original user query, and use those for similarity search.
6. Hyde: Generate hypothetical document to the user query, and use it for similarity search.

## Stages

There are four stages in a RAG pipeline (very similar to an ETL system):

1. Ingestion: Where the pipeline loads the information from the data source.
2. Extraction: Where the pipeline processes the input data and decides how to retrieve the text contained inside them.
3. Transform: Where the pipeline chunks the data and generates document embeddings.
4. Load: Where the pipeline creates a search index in a vector database and loads the document embeddings.

Doing these with unstructured data is not simple:

- You must figure out how to refresh vector databases whenever the original data sources change.
- You must figure out how to extract content from complex documents containing tables, images, or cross-references.
- You must figure out the optimal strategy for chunking your data.

[vectorize](https://vectorize.io/) can help solve these problems.
