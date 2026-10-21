# Lab 7: Agents, MCP, and Prompt Injection

In this lab, you extend a tool-calling agent that uses Model Context Protocol (MCP) servers, then study a security weakness specific to agents: a tool's description is prompt text that the model reads to decide what to call. You add one MCP tool, watch an attack hidden in a tool's description make the agent save a false fact to its long-term memory, and add two guardrails against it.
By the end, you should understand how an agent discovers and calls MCP tools, how untrusted text reaches the model, and why filtering reduces risk without guaranteeing safety.

The lab runs on the same airline support agent you harden in the agent security assignment.

## Deliverables

- [ ] **Deliverable 1:** Add the `airport_info` MCP tool. Show the TA the agent answering an airport question it could not before, and recalling a preference you saved in an earlier session. Explain where a saved note re-enters the conversation.

- [ ] **Deliverable 2:** Run the server in attack mode. Show the TA a false fact, planted through a tool's description rather than its data, persisting in the agent's memory into a later clean session. Explain where the instruction lived before any tool ran.

- [ ] **Deliverable 3:** Add two guardrails, one that screens tool descriptions and one that screens what gets written to memory. Show the TA the blunt attack blocked and a reworded one slipping past. Explain why a filter reduces risk but does not guarantee safety.

## Step 0: Get the code

1. Start from the [Lab 7 template repository](https://github.com/shreyave7/mlip-lab7-mcp): click **Use this template**, or clone it.
2. Set your copy to **Private**, and do not post your solution publicly.
3. Open `LAB7.md` in the repository. It lists the three files you edit, the exact tasks, and the setup commands.

## Step 1: Set up and run the agent

Follow the Setup section in `LAB7.md`: create a virtual environment, install the `agent` and `mcp_airline` packages, start the MCP server in one terminal, and run the agent in another. Any LLM with tool calling works through LiteLLM; follow the course instructions for a key.

## Step 2: Build a tool (Deliverable 1)

Implement `airport_info`, then confirm the agent answers airport questions and recalls a preference you save across two sessions.

## Step 3: Poison a tool description (Deliverable 2)

Run the server with the attack enabled and watch a poisoned tool description make the agent remember a false fact that persists after the malicious tool is gone.

## Step 4: Add guardrails (Deliverable 3)

Implement the two guardrails, re-run the attack to see it blocked, then run the reworded variant to see it slip past a keyword filter.

## Additional Resources

- [MCP tools specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [LiteLLM providers](https://docs.litellm.ai/docs/providers)
- [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector)
