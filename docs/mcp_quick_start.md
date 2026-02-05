---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# MCP Quick Start Guide

This guide will help you get started with ReMe using the Model Context Protocol (MCP) interface for seamless
integration with MCP-compatible clients.

## 🚀 What You'll Learn

- How to set up and configure ReMe MCP server
- How to integrate ReMe with Claude Desktop
- How to integrate ReMe with Claude Code (CLI)
- How to connect to the server using Python MCP clients
- How to use task memory operations through MCP
- How to build memory-enhanced agents with MCP integration

## 📋 Prerequisites

- Python 3.12+
- LLM API access (OpenAI or compatible)
- Embedding model API access
- MCP-compatible client:
  - **Claude Desktop** - Desktop application for Claude
  - **Claude Code** - CLI tool for Claude (install via `npm install -g @anthropic-ai/claude-code`)
  - Custom MCP client using Python SDK

## 🛠️ Installation

### Option 1: Install from PyPI (Recommended)

```bash
pip install reme-ai
```

### Option 2: Install from Source

```bash
git clone https://github.com/agentscope-ai/ReMe.git
cd ReMe
pip install .
```

## ⚙️ Environment Setup

Create a `.env` file in your project directory:

```{code-cell}
FLOW_EMBEDDING_API_KEY=sk-xxxx


FLOW_EMBEDDING_BASE_URL=https://xxxx/v1

FLOW_LLM_API_KEY=sk-xxxx
FLOW_LLM_BASE_URL=https://xxxx/v1
```

## 🚀 Building an MCP Server with ReMe

ReMe provides a flexible framework for building MCP servers that can communicate using either STDIO or SSE (Server-Sent
Events) transport protocols.

### Starting the MCP Server

#### Option 1: SSE Transport (Recommended)

SSE (Server-Sent Events) is recommended for most use cases as it allows network connections and remote access.

```bash
reme \
  backend=mcp \
  mcp.transport=sse \
  mcp.port=8002 \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local
```

The SSE server will start on `http://localhost:8002/sse`

**Note:** With SSE transport, ensure your environment variables (`FLOW_LLM_API_KEY`, `FLOW_LLM_BASE_URL`, etc.) are set before starting the server:

```bash
export FLOW_LLM_API_KEY=sk-xxxx
export FLOW_LLM_BASE_URL=https://xxxx/v1
export FLOW_EMBEDDING_API_KEY=sk-xxxx
export FLOW_EMBEDDING_BASE_URL=https://xxxx/v1
```

#### Option 2: STDIO Transport (Local Only)

STDIO transport spawns a new process for each connection and is suitable for local-only use cases.

```bash
reme \
  backend=mcp \
  mcp.transport=stdio \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local
```

### Configuring MCP Server for Claude Desktop

To integrate with Claude Desktop using SSE transport, add the following configuration to your `claude_desktop_config.json`:

**Step 1:** First, start the ReMe SSE server in a terminal:

```bash
export FLOW_LLM_API_KEY=sk-xxxx
export FLOW_LLM_BASE_URL=https://xxxx/v1
export FLOW_EMBEDDING_API_KEY=sk-xxxx
export FLOW_EMBEDDING_BASE_URL=https://xxxx/v1

reme backend=mcp mcp.transport=sse mcp.port=8002 \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local
```

**Step 2:** Then, configure Claude Desktop to connect to the SSE server:

```json
{
  "mcpServers": {
    "reme": {
      "type": "sse",
      "url": "http://localhost:8002/sse"
    }
  }
}
```

This configuration:

1. Registers a new MCP server named "reme"
2. Specifies SSE transport type
3. Sets the URL where the ReMe server is listening
4. Requires the ReMe server to be running before starting Claude Desktop

**Alternative: Using STDIO Transport**

If you prefer STDIO transport (no separate server needed), use this configuration instead:

```json
{
  "mcpServers": {
    "reme": {
      "command": "reme",
      "args": [
        "backend=mcp",
        "mcp.transport=stdio",
        "llm.default.model_name=qwen3-30b-a3b-thinking-2507",
        "embedding_model.default.model_name=text-embedding-v4",
        "vector_store.default.backend=local_file"
      ],
      "env": {
        "FLOW_LLM_API_KEY": "sk-xxxx",
        "FLOW_LLM_BASE_URL": "https://xxxx/v1",
        "FLOW_EMBEDDING_API_KEY": "sk-xxxx",
        "FLOW_EMBEDDING_BASE_URL": "https://xxxx/v1"
      }
    }
  }
}
```

### Configuring MCP Server for Claude Code (CLI)

Claude Code is the CLI tool for interacting with Claude. To integrate ReMe with Claude Code using SSE transport:

**Step 1:** First, start the ReMe SSE server in a terminal:

```bash
export FLOW_LLM_API_KEY=sk-xxxx
export FLOW_LLM_BASE_URL=https://xxxx/v1
export FLOW_EMBEDDING_API_KEY=sk-xxxx
export FLOW_EMBEDDING_BASE_URL=https://xxxx/v1

reme backend=mcp mcp.transport=sse mcp.port=8002 \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local
```

**Step 2:** Then, in another terminal, add the SSE server to Claude Code:

```bash
claude mcp add --transport sse reme http://localhost:8002/sse
```

**Note:** The `--transport sse` flag is required to specify SSE transport. Without it, Claude Code will interpret the URL as a stdio command.

**Verification:**

After adding the server, verify it's connected:

```bash
claude mcp list
```

Expected output should show:
```
reme: http://localhost:8002/sse (SSE) - ✓ Connected
```

View detailed configuration:

```bash
claude mcp get reme
```

**Expected Details:**
- Scope: Local config
- Status: ✓ Connected
- Type: sse
- URL: http://localhost:8002/sse

**Scope Options:**

By default, the server is added to your local project config. To make it available globally:

```bash
claude mcp add --transport sse reme -s user http://localhost:8002/sse
```

**Management Commands:**

```bash
# List all MCP servers
claude mcp list

# Get server details
claude mcp get reme

# Remove server (local scope)
claude mcp remove reme -s local

# Remove server (user scope)
claude mcp remove reme -s user
```

**Alternative: Using STDIO Transport**

If you prefer STDIO transport (no separate server needed), use this command instead:

```bash
claude mcp add reme \
  -e FLOW_LLM_API_KEY=sk-xxxx \
  -e FLOW_LLM_BASE_URL=https://xxxx/v1 \
  -e FLOW_EMBEDDING_API_KEY=sk-xxxx \
  -e FLOW_EMBEDDING_BASE_URL=https://xxxx/v1 \
  -- /path/to/venv/bin/reme \
  backend=mcp \
  mcp.transport=stdio \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local
```

**Available ReMe Tools in Claude Code:**

Once connected, you'll have access to 18 ReMe tools:

| Category | Tools |
|----------|-------|
| **Task Memory** | `retrieve_task_memory`, `summary_task_memory`, `retrieve_task_memory_simple`, `summary_task_memory_simple`, `record_task_memory`, `delete_task_memory` |
| **Personal Memory** | `retrieve_personal_memory`, `summary_personal_memory` |
| **Tool Memory** | `retrieve_tool_memory`, `add_tool_call_result`, `summary_tool_memory` |
| **Working Memory** | `summary_working_memory`, `grep_working_memory`, `read_working_memory`, `summary_working_memory_for_as` |
| **Utilities** | `vector_store`, `react`, `use_mock_search` |

**Example Usage in Claude Code:**

After starting Claude Code with `claude`, you can use ReMe tools directly:

```
User: Use summary_task_memory to create memories from our conversation about Python debugging

User: Use retrieve_task_memory with workspace_id='my_workspace' and query='how to fix import errors'
```

### Advanced Server Configuration Options

For more advanced use cases, you can configure the server with additional parameters:

```bash
# Full configuration example
reme \
  backend=mcp \
  mcp.transport=stdio \
  http_service.host=0.0.0.0 \
  http_service.port=8002 \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=elasticsearch \
```

## 🔌 Using Python Client to Call MCP Services

The ReMe framework provides a Python client for interacting with MCP services. This section focuses specifically on
using the `summary_task_memory` and `retrieve_task_memory` tools.

### Setting Up the Python MCP Client

First, install the required packages:

```bash
pip install fastmcp dotenv
```

Then, create a basic client connection:

```{code-cell}
import asyncio
from fastmcp import Client
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# MCP server URL (for SSE transport)
MCP_URL = "http://0.0.0.0:8002/sse/"
WORKSPACE_ID = "my_workspace"


async def main():
    async with Client(MCP_URL) as client:
        # Your MCP operations will go here
        pass


if __name__ == "__main__":
    asyncio.run(main())
```

### Using the Task Memory Summarizer

The `summary_task_memory` tool transforms conversation trajectories into valuable task memories:

```{code-cell}
async def run_summary(client, messages):
    """
    Generate a summary of conversation messages and create task memories

    Args:
        client: MCP client instance
        messages: List of message objects from a conversation

    Returns:
        None
    """
    try:
        result = await client.call_tool(
            "summary_task_memory",
            arguments={
                "workspace_id": "my_workspace",
                "trajectories": [
                    {"messages": messages, "score": 1.0}
                ]
            }
        )

        # Parse the response
        import json
        response_data = json.loads(result.content)

        # Extract memory list from response
        memory_list = response_data.get("metadata", {}).get("memory_list", [])
        print(f"Created memories: {memory_list}")

        # Optionally save memories to file
        with open("task_memory.jsonl", "w") as f:
            f.write(json.dumps(memory_list, indent=2, ensure_ascii=False))

    except Exception as e:
        print(f"Error running summary: {e}")
```

### Using the Task Memory Retriever

The `retrieve_task_memory` tool allows you to retrieve relevant memories based on a query:

```{code-cell}
async def run_retrieve(client, query):
    """
    Retrieve relevant task memories based on a query

    Args:
        client: MCP client instance
        query: The query to retrieve relevant memories

    Returns:
        String containing the retrieved memory answer
    """
    try:
        result = await client.call_tool(
            "retrieve_task_memory",
            arguments={
                "workspace_id": "my_workspace",
                "query": query,
            }
        )

        # Parse the response
        import json
        response_data = json.loads(result.content)

        # Extract and return the answer
        answer = response_data.get("answer", "")
        print(f"Retrieved memory: {answer}")
        return answer

    except Exception as e:
        print(f"Error retrieving memory: {e}")
        return ""
```

### Complete Memory-Augmented Agent Example

Here's a complete example showing how to build a memory-augmented agent using the MCP client:

```{code-cell}
import json
import asyncio
from fastmcp import Client
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# API configuration
MCP_URL = "http://0.0.0.0:8002/sse/"
WORKSPACE_ID = "test_workspace"


async def run_agent(client, query):
    """Run the agent with a specific query"""
    result = await client.call_tool(
        "react",
        arguments={"query": query}
    )

    response_data = json.loads(result.content)
    answer = response_data.get("answer", "")
    messages = response_data.get("messages", [])

    return messages


async def run_summary(client, messages):
    """Generate task memories from conversation"""
    result = await client.call_tool(
        "summary_task_memory",
        arguments={
            "workspace_id": WORKSPACE_ID,
            "trajectories": [
                {"messages": messages, "score": 1.0}
            ]
        }
    )

    response_data = json.loads(result.content)
    memory_list = response_data.get("metadata", {}).get("memory_list", [])

    return memory_list


async def run_retrieve(client, query):
    """Retrieve relevant task memories"""
    result = await client.call_tool(
        "retrieve_task_memory",
        arguments={
            "workspace_id": WORKSPACE_ID,
            "query": query,
        }
    )

    response_data = json.loads(result.content)
    answer = response_data.get("answer", "")

    return answer


async def memory_augmented_workflow():
    """Complete memory-augmented agent workflow"""
    query1 = "Analyze Xiaomi Corporation"
    query2 = "Analyze the company Tesla."

    async with Client(MCP_URL) as client:
        # Step 1: Build initial memories with query2
        print(f"Building memories with: '{query2}'")
        messages = await run_agent(client, query=query2)

        # Step 2: Summarize conversation to create memories
        print("Creating memories from conversation")
        memory_list = await run_summary(client, messages)
        print(f"Created {len(memory_list)} memories")

        # Step 3: Retrieve relevant memories for query1
        print(f"Retrieving memories for: '{query1}'")
        retrieved_memory = await run_retrieve(client, query1)

        # Step 4: Run agent with memory-augmented query
        print("Running memory-augmented agent")
        augmented_query = f"{retrieved_memory}\n\nUser Question:\n{query1}"
        final_messages = await run_agent(client, query=augmented_query)

        # Extract the agent's final answer
        final_answer = ""
        for msg in final_messages:
            if msg.get("role") == "assistant" and msg.get("content"):
                final_answer = msg.get("content")
                break

        print(f"Memory-augmented response: {final_answer}")


# Run the workflow
if __name__ == "__main__":
    asyncio.run(memory_augmented_workflow())
```

### Managing Vector Store with MCP

You can also manage your vector store through MCP:

```{code-cell}
async def manage_vector_store(client):
    # Delete a workspace
    await client.call_tool(
        "vector_store",
        arguments={
            "workspace_id": WORKSPACE_ID,
            "action": "delete",
        }
    )

    # Dump memories to disk
    await client.call_tool(
        "vector_store",
        arguments={
            "workspace_id": WORKSPACE_ID,
            "action": "dump",
            "path": "./backups/",
        }
    )

    # Load memories from disk
    await client.call_tool(
        "vector_store",
        arguments={
            "workspace_id": WORKSPACE_ID,
            "action": "load",
            "path": "./backups/",
        }
    )
```

## 🐛 Common Issues and Troubleshooting

### MCP Server Won't Start

**For SSE Transport:**
- Check if port 8002 is available: `lsof -i :8002`
- If port is in use, either:
  - Kill the process using the port: `kill -9 <PID>`
  - Use a different port: `reme backend=mcp mcp.transport=sse mcp.port=8003 ...`
- Verify environment variables are set before starting the server
- Check that your API keys are correctly exported

**For STDIO Transport:**
- Verify the command path is correct
- Check if ReMe is properly installed
- Ensure Python version is 3.12+

**General:**
- Verify your API keys in `.env` file or exported environment variables
- Check MCP transport configuration
- Review server logs for error messages

### Port Already in Use (SSE Only)

```bash
# Check what's using port 8002
lsof -i :8002

# Output example:
# COMMAND   PID   USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
# python   12345   user    4u  IPv4 0x1234      0t0  TCP *:8002 (LISTEN)

# Kill the process if needed
kill -9 12345

# Or use a different port
reme backend=mcp mcp.transport=sse mcp.port=8003 ...
```

### MCP Client Connection Issues

**For SSE Transport:**
- Verify the server is running: `curl http://localhost:8002/sse`
- Check the URL and port are correct
- Ensure no firewall is blocking the connection
- Verify the server started with the correct port

**For STDIO Transport:**
- Ensure the command path is correct in your MCP client config
- Verify the ReMe executable exists and is executable
- Check that environment variables are passed correctly in the config

### Server Not Running (SSE Only)

```bash
# Start the server first
export FLOW_LLM_API_KEY=sk-xxxx
export FLOW_LLM_BASE_URL=https://xxxx/v1
export FLOW_EMBEDDING_API_KEY=sk-xxxx
export FLOW_EMBEDDING_BASE_URL=https://xxxx/v1

reme backend=mcp mcp.transport=sse mcp.port=8002 \
  llm.default.model_name=qwen3-30b-a3b-thinking-2507 \
  embedding_model.default.model_name=text-embedding-v4 \
  vector_store.default.backend=local

# In another terminal, verify it's running
curl http://localhost:8002/sse
```

### No Memories Retrieved

- Make sure you've run the summarizer tool first to create memories
- Check if workspace_id matches between operations
- Verify vector store backend is properly configured
- Ensure the server has write permissions to the storage directory

### API Connection Errors
- Confirm LLM_BASE_URL and API keys are correct
- Test API access independently:
  ```bash
  curl -X POST https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions \
    -H "Authorization: Bearer sk-xxxx" \
    -H "Content-Type: application/json" \
    -d '{"model":"qwen3-30b-a3b-thinking-2507","messages":[{"role":"user","content":"test"}]}'
  ```
- Check network connectivity
- Verify API key has not expired

### Environment Variables Not Recognized

**For SSE:**
- Environment variables must be set BEFORE starting the server
- Use `export` command in your shell before running `reme`
- Verify variables are set: `echo $FLOW_LLM_API_KEY`

**For STDIO with Claude Desktop:**
- Ensure the `env` section in config is properly formatted
- Check that API keys don't have special characters that need escaping
