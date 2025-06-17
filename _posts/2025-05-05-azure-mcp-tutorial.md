---
title: "Azure MCP Tutorial"
categories:
  - blog
tags:
  - Azure
  - Agents
  - AI
---

## Mastering Model Context Protocol(MCP): Building MultiServer MCP with Azure OpenAI

The Model Context Protocol (MCP) is rapidly becoming the prominent framework for building truly agentic, interoperable AI applications. While there are several MCP templates online, this project stands out as the only starter template that combines **Azure OpenAI integration** with a **Multi-Server MCP** architecture on a custom interface, enabling you to connect and orchestrate multiple tool servers—through a customized user interface.

Many articles document MCP servers for single-server use, this project covers Multiserver MCP implementation, connect multiple MCP servers in a single client session through MultiServerMCP library from Langchain adapters, enabling agentic orchestration across different domains. While most triggers to OOB MCP servers leveraged Github Copilot for input, this project allows for custom app integration

In this technical blog, we’ll walk through the architecture, components, and step-by-step code to build your own MultiServer MCP using the Langchain MCP adapter, FastAPI, and a custom frontend which also lists the loaded MCP tools. By the end, you’ll have a scalable, extensible agentic platform ready for expanding your real-world AI applications.

---
### Quick recap: Why is MCP important?

![alt text](/assets/images/2025-05-05/1.png)
The Model Context Protocol (MCP) is important because it provides a standardized way for agents and AI applications to interact with a wide variety of tools and services, regardless of the programming language or platform. Think of it as the universal connector for AI agents.

### Why Multi Server MCP?

As solutions evolve in the real-world and we approach complex workflows, we often need several specialised features – think scraping the web, querying a database, and editing files, all in one agent.

- **Plug-and-play tool servers:** Connect custom and ready-made MCP servers (e.g., custom Math, custom Fabric, Azure AI Foundry, Azure, Claude).
- **Interoperability:** Different AI tools can work together seamlessly.
- **Agentic orchestration:** Enable your agent to reason, plan, and call tools across multiple specialized domains
- **Azure OpenAI integration:** Leverage enterprise-grade LLMs and security.
- **Scalability:** Easily add or remove tool servers without changing the agent code.

---

## Architecture Overview

**Key Components:**

- **MCP Server:** A process exposing tools (functions) via the MCP protocol (stdio, TCP, etc.).
- **MCP Client:** Connects to one or more MCP servers, discovers tools, and enables agentic orchestration.
- **Langchain MCP Adapter:** Bridges Langchain agents with MCP tools.
- **FastAPI Backend:** Hosts the API and serves the frontend.
- **Frontend:** Simple HTML/JS UI for user interaction.

**Benefits:**  

- Decoupled, language-agnostic tool servers  
- Dynamic tool discovery  
- Seamless integration with LLMs and external APIs  
- Scalable and secure (especially with Azure OpenAI)

**Considerations:**

- Scaling is tricky, MCP exposed through SSE each requires a unique dedicated port exposed, which can create scalability challenges (here I'm using stdio for ease of startup).
- Tool discovery is limited to the MCP servers registered in the client configuration.
- Requires careful management of tool namespaces to avoid conflicts.
- MCP client requires re-initialization to detect new servers or tools, challenge during runtime discovery.
- Proper security definition for custom MCP Servers are still lacking.

---

## Step-by-Step Guide: Building a Multi MCP Server Application

![alt text](/assets/images/2025-05-05/2.png)

### 1. Import Langchain and MultiServer MCP

```python
from langchain_openai import AzureChatOpenAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent
```

### 2. Create a FastAPI Frontend Application

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse, FileResponse
import uvicorn

app = FastAPI()

@app.post("/ask")
async def ask(request: Request):
    data = await request.json()
    message = data.get("message", "")
    result = await run_agent(message)
    return JSONResponse(content={"result": result})

@app.get("/")
async def root():
    return FileResponse("frontend.html", media_type="text/html")
```

### 3. Create a Custom MCP Math Server

```python
# math_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Math")

@mcp.tool(name="Addition", description="Adds two integers")
def add(a: int, b: int) -> int:
    return a + b

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 4. Import and Register Azure AI Foundry MCP Server

Add the Azure AI Foundry server to your `.vscode/mcp.json`:

```jsonc
{
  "servers": {
    "mcp_foundry_server": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--prerelease=allow",
        "--from",
        "git+https://github.com/azure-ai-foundry/mcp-foundry.git",
        "run-azure-ai-foundry-mcp",
        "--envFile",
        "${workspaceFolder}/.env"
      ]
    }
  }
}
```

In your Python code, dynamically load all servers:

```python
import json, os

MCP_CLIENT_CONFIG = {
    "math": { ... },  # as above
    "fabric": { ... }
}
with open(os.path.join(".vscode", "mcp.json"), "r") as f:
    mcp_config = json.load(f)
for name, cfg in mcp_config["servers"].items():
    args = [arg.replace("${workspaceFolder}", os.getcwd()) for arg in cfg["args"]]
    MCP_CLIENT_CONFIG[name] = {
        "command": cfg["command"],
        "args": args,
        "transport": cfg["type"]
    }
client = MultiServerMCPClient(MCP_CLIENT_CONFIG)
```

### 5. Show the Tools List

```python
async def get_tools():
    async with client:
        tools = client.get_tools()
        return [{"name": t.name, "description": getattr(t, "description", "")} for t in tools]
```

### 6. Final App: Agentic Orchestration

```python
async def run_agent(message):
    async with client:
        tools = client.get_tools()
        agent = create_react_agent(model, tools)
        agent_response = await agent.ainvoke({"messages": message})
        return agent_response["messages"][3].content
```

---

## Bonus: Tools Endpoint for Frontend Discovery

A `/tools` endpoint was created in the FastAPI backend to allow the frontend to display all connected MCP tools in real time:

```python
@app.get("/tools")
async def get_tools():
    async with client:
        tools = client.get_tools()
        return {"tools": [
            {"name": t.name, "description": getattr(t, "description", "")}
            for t in tools
        ]}
```

---

## Conclusion

With this setup, you can:

- Effortlessly integrate both custom and pre-built MCP servers (such as Math, Fabric, Azure AI Foundry, and more)
- Develop a tailored FastAPI frontend for interactive user experiences
- Instantly discover and orchestrate tools from all connected servers
- Harness the capabilities of Azure OpenAI and other LLMs within a secure, scalable, and agentic framework

**Ready to build your own agentic platform? [Fork this repo](https://github.com/delynchoong/azure-openai-agent-multi-mcp-starter) and start connecting your own MCP tools today!**

## Additional Notes

If you prefer, you can also leverage virtual environment to store your environment variables.
Suppose your project, Project A, is written against a specific version of library X. In the future, you might need to upgrade library X. Say, for example, you need the latest version for another project you started, called Project B. You upgrade library X to the latest version, and project B works fine. Great! But once you did this, it turns out your Project A code broke badly. After all, APIs can change significantly on major version upgrades.

A virtual environment fixes this problem by isolating your project from other projects and system-wide packages. You install packages inside this virtual environment specifically for the project you are working on.

Alternatively, this can be achieved using Github Codespaces.

