# AI Engineering

The central purpose of the AI Engineer is to **turn existing foundation model capabilities into useful products**. "Useful products" will have utility, entertainment, or creative value, and is something we will never have enough of.

## Generative AI Product

- System takes in a question and decides which AI agent is best equipped to handle it
- AI agent gathers information via a combination of internal APIs
- AI agent crafts a response with the necessary information in hand. It filters and synthesizes the data intoa coherent, informative answer. To avoid generating a wall of ext and make the experience more interactive, internal APIs are invoked to decorate the response with attachments like article links or profiles.

### Design

1. **Routing**: Decides if query is in scope or not, and which AI agent to forward it to
2. **Retrieval**: Recall-oriented step where AI agent decides which services to call and how
3. **Generation**: Precision-oriented step that sieves through the noisy data retrieved, filters it and produces the final response

- Tuning routing and retrieval can involve prompt engineering and in-house models.
- Generation may require a lot of work if 99%+ of answers should be great.
- Small models for routing/retrieval, bigger models for generation
- Embedding-Based Retrieval (EBR) powered by an in-memory database as "poor man's fine tuning" to inject response examples directly into prompts
- Per-step specific evaluation pipelines, particularly for routing/retrieval

## Open Questions

- Is it necessary to have RAG with vector index in front of chat conversation calls to a generative AI provider?

## Links

- [deeplearning.ai](https://www.deeplearning.ai/)
- [Vercel AI SDK](https://github.com/vercel/ai)
- [GitHub Models](https://github.blog/news-insights/product-news/introducing-github-models/)
- [Building a a Generative AI Product](https://www.linkedin.com/blog/engineering/generative-ai/musings-on-building-a-generative-ai-product)
- [/r/singularity](https://old.reddit.com/r/singularity/)
