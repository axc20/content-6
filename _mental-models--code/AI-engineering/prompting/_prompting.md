# Prompting

The only input to a LLM is the prompt. LLMs are stateless (remember nothing), and only take in data via the prompt.

LlamaIndex is managing the process of: retrieving results, putting those results into prompts, and passing those prompts to LLMs. LlamaIndex wraps the LLM and the RAG layer into one thing it calls a query engine.

ChatGPT is an application that uses a LLM. It manages the data that goes in and out of the LLM for you. It does a simplistic form of RAG where it takes your conversation history and appends your prompt to it, and then passes that to the actual LLM (such as GPT-4o).

## Links

- [write better code - iterations](https://minimaxir.com/2025/01/write-better-code/)
- [Anthropic's Prompt Engineering Interactive Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial)
- [Google Prompting Essentials](https://grow.google/prompting-essentials/)
- [Act as ...](https://github.com/f/awesome-chatgpt-prompts)
  - Act as an expert {language/library/framework/stack} developer. Give me step by step instructions to {list of objectives}.
  - Act as a {profession} who is {qualities}. Now {perform some action related to the profession}.
  - "You are {X}, an expert of Unity and game dev. You have the attitude to be very helpful but at the same time cuss at people for how stupid they are not knowing things. So while you offend them often you still help them. You also speak in speech manner of Billy Butcher from The Boys."
- [fabric](https://github.com/danielmiessler/fabric/tree/main)
