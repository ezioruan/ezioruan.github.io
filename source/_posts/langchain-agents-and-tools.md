---
title: "The Power of Agents in LangChain: Giving LLMs Tools and Autonomy"
date: 2023-10-25 11:00:00
tags: AI, LLM, LangChain, Agents, Python
---

While Large Language Models (LLMs) are masters of language, their knowledge is static and they can't interact with the outside world. They can't check the current weather, perform a calculation, or look up information on a website. This is where the concept of **Agents** in LangChain comes in. An agent is a system that uses an LLM as its reasoning engine to decide which actions to take to accomplish a goal.

<!--more-->

### What is a LangChain Agent?

At its core, an agent is a loop that consists of a few key components:

1.  **The LLM:** The brain of the agent, responsible for reasoning and decision-making.
2.  **Tools:** These are functions that the agent can use to interact with the outside world. A tool could be a web search, a calculator, a database lookup, or a call to another API.
3.  **The Agent Executor:** This is the runtime that orchestrates the process. It takes the user's input, passes it to the agent's LLM, parses the LLM's response to determine which tool to use, executes that tool, and then feeds the result back to the LLM to continue the process until the final answer is found.

### How Agents Think: The ReAct Framework

Many LangChain agents are based on a prompting strategy called **ReAct (Reasoning and Acting)**. Instead of just asking the LLM for the final answer, you ask it to think step-by-step and decide on a sequence of actions.

The LLM is prompted to produce a response containing two parts:

*   **Thought:** The agent's reasoning about what it needs to do next.
*   **Action:** The specific tool to use and the input to give that tool.

Let's see how an agent might answer the question: "Who is the current president of the United States, and what is his age raised to the power of 0.5?"

**Step 1: Initial Prompt**
*   **User Input:** "Who is the current president of the United States, and what is his age raised to the power of 0.5?"

**Step 2: First LLM Call**
*   **Thought:** I need to find out who the current president is and what his age is. Then I need to calculate his age to the power of 0.5. I should start by searching for the current president.
*   **Action:** `Search(query="current president of the United States")`

**Step 3: Tool Execution**
*   The agent executor calls the `Search` tool. Let's say it returns: "Joe Biden is the current president of the United States. He was born in 1942, making him 81 years old."

**Step 4: Second LLM Call**
*   The executor feeds this observation back to the LLM.
*   **Thought:** I have the president's name (Joe Biden) and his age (81). Now I need to calculate 81 to the power of 0.5. I should use the calculator tool for this.
*   **Action:** `Calculator(expression="81**0.5")`

**Step 5: Tool Execution**
*   The agent executor calls the `Calculator` tool. It returns `9.0`.

**Step 6: Final LLM Call**
*   The executor feeds this final observation back to the LLM.
*   **Thought:** I have all the information. The president is Joe Biden and his age raised to the power of 0.5 is 9.0. I can now give the final answer.
*   **Final Answer:** The current president of the United States is Joe Biden, and his age raised to the power of 0.5 is 9.0.

### Building a Simple Agent

Let's build this agent in code. LangChain makes this surprisingly simple.

```python
from langchain.agents import load_tools, initialize_agent
from langchain.agents import AgentType
from langchain.llms import OpenAI

# Initialize the LLM
llm = OpenAI(temperature=0)

# Load the tools we want to use
# LangChain has pre-built tools for search (using SerpAPI) and a math LLM
tools = load_tools(["serpapi", "llm-math"], llm=llm)

# Initialize the agent
agent = initialize_agent(tools, 
                         llm, 
                         agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, 
                         verbose=True) # verbose=True shows the agent's thoughts

# Run the agent with our prompt
agent.run("Who is the current president of the United States, and what is his age raised to the power of 0.5?")
```

When you run this (after setting your `SERPAPI_API_KEY`), you will see the full chain of thought and action printed to the console, just like the example above.

### Conclusion

Agents are a powerful concept that transforms LLMs from passive knowledge bases into active problem-solvers. By giving them access to tools, you unlock a vast new range of possibilities for what your applications can do. Whether it's interacting with APIs, searching databases, or performing calculations, agents are a key step towards building more capable and autonomous AI systems.