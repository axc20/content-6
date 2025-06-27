# RAG: Retrieval Augmented Generation

1. Custom knowledge base
   - Collection of relevant and up-to-date information that serves as a foundation for RAG
   - Can be a database, a set of documents, or a combination of both
2. Chunking
   - Process of breaking down a large input text into smaller pieces
   - This ensures that text fits the input size of the embedding model and improves retrieval efficiency
   - There should be some overlapping for the chunks
3. Embeddings and embedding model
4. Vector databases
   - A collection of pre-computed vector representations of text data for fast retrieval and similarity search, with capabilities like CRUD operations, metadata filtering, and horizontal scaling
5. User chat interface
   - A user-friendly interface that allows users to interact with the RAG system, providing input query and receiving output
   - The query is converted to an embedding which is used to retrieve relevant context from the vector DB
6. Prompt template
   - Process of generating a suitable prompt for the RAG system, which can be a combination of the user query and the custom knowledge base
   - This is given as an input to an LLM that produces the final response

## Contextual Retrieval

This is a technique to reudce the error rate of RAG by adding important context to small text chunks before storing them. For example, instead of saying "the company grew by 3%", include details like which company and when.

## Resources

- pypdf
- faiss-gpu

## Links

- [RAG using Llama 3.1](https://lightning.ai/lightning-ai/studios/rag-using-llama-3-1-by-meta-ai?utm_source=akshay)
