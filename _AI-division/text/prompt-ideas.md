# Prompt Ideas

2. Write pseudocode to solve {problem}, explaining your reasoning.

## Examples

- I would like to incubate a startup founded by GPT4. Please write instructions and code for a code that runs intermittently to create, complete, and prioritize tasks. This should use Python and GPT4 API, and store tasks in Pinecone. Use Langchain agents with ChatGPT plug-ins to complete tasks. Each loop should consist of the folloing steps: (1) review task list and complete first task by retrieving relevant context from Pinecone and using Langchain agent with ChatGPT plug-ins, (2) enrich result of task with further context and store in Pinecone, (3) create new tasks based on results, add to task list, and re-prioritize task list for next iteration.
