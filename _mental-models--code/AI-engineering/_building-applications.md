## Frameworks

- [Langchain](https://github.com/langchain-ai/langchain)
  - Allows users to chain together different components, such as data sources, APIs, and language models to create more complex and adaptive applications. For example, a chatbot that can answer questions based on data from web scraping, databases, or cloud storage.
  - LangChain can also help users generate prompts for few-shot learning, which is a meta-learning technique that allows LLMs to learn new tasks from a few examples.

## Vector Databases

- QDrant
- pgvector
  - Timescale pgai Vectorizer - lets you create, sync, and manage embeddings with just one SQL command
- [sqlite-vec](https://github.com/asg017/sqlite-vec)
- Pinecone
- Weaviate
- Chroma

Vector databases allow you to easily index and store large amounts of information, and then quickly query for similar pieces of information to give your model when you need to.

Developer libraries like Langchain and LlamaIndex wrap some business logic around the database layer to make it easier for AI developers to do common tasks. They make it easier to make AI apps by reducing the amount of boilerplate code being written to take a PDF or webpage with interesting information on it, parse it, break it into chunks, store it, and retrieve it for use in AI apps.

**Private repositories of knowledge are going to be very valuable.** It is expensive and time consuming to find information that's relevant to the things you think about, so there is great value in collecting and curating your own personal set of notes, articles, books, and highlights. From this knowledge base, you can customize your AI experience so it's more useful to you right off the bat.
