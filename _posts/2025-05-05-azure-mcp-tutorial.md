---
title: "Mastering Model Context Protocol(MCP): Building MultiServer MCP with Azure OpenAI"
categories:
  - blog
tags:
  - Azure
  - Agents
  - AI
---

## Mastering Model Context Protocol(MCP): Building MultiServer MCP with Azure OpenAI

The [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction?wt.mc_id=studentamb_263805) is rapidly becoming the prominent framework for building truly agentic, interoperable AI applications.

Many articles document MCP servers for single-server use, this project stands out as the starter template that combines **Multi-Server MCP** with Azure OpenAI integration on a custom interface, enabling you to connect and orchestrate multiple tool servers—through a customized UI.

Here, we deep dive into Multiserver MCP implementation, connecting both local custom and ready MCP Servers in a single client session through the MultiServerMCP library from Langchain adapters, enabling agentic orchestration across different domains. While most triggers to OOB MCP servers leveraged Github Copilot for input, this project allows for custom app integration

In this technical blog, we’ll walk through the architecture, components, and step-by-step code to build your own MultiServer MCP using the Langchain MCP adapter, FastAPI, and a custom frontend which also lists the loaded MCP tools. By the end, you’ll have a scalable, extensible agentic platform ready for expanding your real-world AI applications.

---

### Quick recap: Why is MCP important?

![alt text](/assets/images/2025-05-05/1.png)
The Model Context Protocol (MCP) provides a standardized way for agents and AI applications to interact with a wide variety of tools and services, regardless of the programming language or platform. Think of it as the universal connector for AI
applications.

There are now several ready MCP servers available, such as [Azure AI Foundry](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-mcp-plugins?pivots=programming-language-python) and [Azure MCP Server](https://github.com/Azure/azure-mcp), providing a rich ecosystem of existing tools that can be easily integrated into your applications.

### Why Multi Server MCP?

As solutions evolve in the real-world and we approach complex workflows, we often need several specialised APIs, here, think of it as tools – functions like querying a database, editing files, getting information from websites - available all in one agent. MultiServer MCP allows you to connect and orchestrate these diverse tools seamlessly:

- **Plug-and-play tool servers:** Connect custom. remote and ready-made MCP servers (e.g., custom Math, custom Fabric, Azure AI Foundry, Azure, Claude).
- **Interoperability:** Different AI tools can work together seamlessly.
- **Agentic orchestration:** Enable your agent to reason, plan, and call tools across multiple specialized domains without defined routing.
- **Azure OpenAI integration:** Leverage enterprise-grade LLMs and security.
- **Scalability:** Easily add or remove tool servers without changing the agent code.

---

## Architecture Overview

![alt text](/assets/images/2025-05-05/4.png)

**Key Components:**

- **MCP Server:** A process exposing tools (functions) via the MCP protocol (stdio, SSE, etc.).
- **MCP Client:** Connects to one or more MCP servers, discovers tools, and enables agentic orchestration.
- **Langchain MCP Adapter:** Bridges Langchain agents with MCP tools.
- **FastAPI Backend:** Hosts the API and serves the frontend.

**Benefits:**  

- Decoupled, language-agnostic tool servers  
- Dynamic tool discovery  
- Seamless integration with LLMs and external APIs  
- Scalable and secure (especially with Azure OpenAI)

## Key Files in This Project

| File Name            | Purpose                                                      |
|----------------------|--------------------------------------------------------------|
| client.py            | Main client logic: connects to multiple MCP servers, handles tool discovery, and agent orchestration |
| frontend_api.py      | FastAPI backend serving the custom frontend and API endpoints |
| math_server.py       | Example custom MCP server exposing math tools                |
| .vscode/mcp.json     | Configuration for registering MCP servers (local/remote)     |
| frontend.html        | Custom HTML/JS frontend for user interaction                 |

---

## Step-by-Step Guide: Building a Multi MCP Server Application

![alt text](/assets/images/2025-05-05/2.png)



### 1. Import Langchain and MultiServer MCP in client.py file

```python
from langchain_openai import AzureChatOpenAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent
```

### 2. Create a FastAPI Frontend Application in frontend_api.py

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

Add the Azure AI Foundry server to your `.vscode/mcp.json`. Refer here for more configurations setting up the [Azure AI Foundry MCP Server](https://github.com/azure-ai-foundry/mcp-foundry).

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
### 
**Considerations:**

- Scaling is tricky, enterprise-ready MCP would be better exposed through SSE, however each endpoint requires a unique dedicated port exposed, which can create scalability challenges (here I'm using stdio for ease of startup).
- Tool discovery is limited to the MCP servers registered in the client configuration.
- Requires careful management of tool namespaces to avoid conflicts.
- MCP client requires re-initialization to detect new servers or tools, challenge during runtime discovery.
- Proper security definition for custom MCP Servers are still lacking.
  
**Recommendation:**
- - ![alt text](/assets/images/2025-05-05/3.png)
- Multi Server MCP and enterprise-level solutions should leverage [APIM as auth gateway to MCP](https://techcommunity.microsoft.com/blog/integrationsonazureblog/azure-api-management-your-auth-gateway-for-mcp-servers/4402690) and [MCP Plugin for Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-mcp-plugins?pivots=programming-language-python).
- If you prefer, you can also use virtual environments to store your environment variables.
