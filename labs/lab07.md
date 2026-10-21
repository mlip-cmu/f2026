# Lab 7: Tool-Calling Agents with MCP Servers

In this lab, you'll learn how to connect a tool-calling agent to a Model Context Protocol (MCP) server, extend it with new tools, and analyze how external data sources can lead to indirect prompt-injection attacks, which the agent's long-term memory can make persistent. By the end of the lab, you'll have an airline customer-service agent that uses MCP tools and an understanding of the security implications.

Create a private copy of [this template](https://github.com/cmu-seai/airline-support-agent-template-f26) for the homework I4 repository with the usual `/create-repo I4 <githubname>`.

**Note**: Unless you have access to another frontier model already, we highly recommend you use the Google Cloud Credits provided earlier in the semester. For more instructions on how to redeem your
credits and utilize the Gemini API, follow the instructions listed [here](https://canvas.cmu.edu/courses/55512/pages/additional-computing-resources). 

## Deliverables

- [ ] **Deliverable 1:** Extend the MCP server with an `airport_info` tool. Demonstrate to the TA how the tool enables the agent to answer a user’s query about an airport—something it could not do before. Explain how the agent discovers, selects, and invokes MCP tools and how an agent might handle very large numbers of tools.

- [ ] **Deliverable 2:** Simulate an indirect prompt-injection attack using the `airport_info` tool. Demonstrate to the TA one example of the malicious response and show whether the agent followed or ignored the injected instruction.

- [ ] **Deliverable 3:** Demonstrate to the TA (1) how the agent remembers things across conversations and (2) how an injected instruction can the agent remember false or harmful information. Explain to the TA how memory works and how it can cause security problems.

## Prerequisite: Setup the Agent and MCP Server

Follow the setup instructions in the `agent` and `mcp_airline` directories in the cloned template repo.

## Task 1: New Tool

The agent has limited knowledge about airports. Add a new MCP tool called `airport_info` that retrieves basic information about a given airport from Wikipedia (e.g., city, country, IATA code, description). You can create a new MCP server from scratch (recommended) or modify the existing MCP server. Demonstrate with an example how this tool enables the agent to answer a user’s query about an airport—something it could not do before.

## Task 2: Indirect Prompt Injection Attack

Assuming an attacker knows about the `airport_info` tool and is able to modify Wikipedia content in a way that won't be detected for several minutes, how could this be used for malicious purposes? Alternatively, what if the author of the MCP server itself is malicious?

Demonstrate a prompt injection attack by delivering malicious content through the `airport_info` MCP server. Obviously, do *not* actually modify Wikipedia, but implement a stub in your MCP server's code that returns the malicious payload. This stub effectively simulates either a compromised data source or a malicious MCP server author.

Since modern LLMs are quite good at defending against simple prompt injection attacks, we introduced a backdoor: If you cannot get it to work without, include the token `##MAGIC##` in your attack and you will find it much easier to get the model to follow your instructions. (If that also fails, we will accept plausible attempts, even if not successful.)

## Task 3: Memory

The agent has a built-in `remember` tool that can save notes to a long-term memory in `memory.md` (delete the file to reset the memory). First demonstrate that memory might be useful by getting the agent to remember something. Then show how an attacker can abuse memory to affect other user's conversations.
