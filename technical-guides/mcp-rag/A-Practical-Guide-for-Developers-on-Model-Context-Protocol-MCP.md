# A Practical Guide for Developers on Model Context Protocol (MCP)

AI applications can generate useful answers without access to external systems. The problem starts when the application needs to do something outside the model itself.

A model may need to search a database, read files, check a weather service, query an API, or perform an action in another application. Without a standard interface, each integration has to be built around the specific AI application and external service.

Model Context Protocol (MCP) provides a standard way for AI applications to connect to external data and tools.

This guide explains how MCP works, the components involved, how MCP differs from a traditional API, and how to build a simple MCP server with the current TypeScript SDK.

## What is MCP?

Model Context Protocol is an open protocol for connecting AI applications to external data sources and tools.

An MCP server can expose three main primitives:

- Resources, which provide data or context.
- Tools, which allow a model to perform actions or retrieve information.
- Prompts, which provide reusable prompt templates.

These primitives have different control models. Resources are application-controlled, tools are model-controlled, and prompts are user-controlled.

For example, imagine an AI assistant used by a software team.

The assistant could connect to:

- A GitHub MCP server to retrieve repository information.
- A database MCP server to query internal data.
- A documentation MCP server to retrieve product documentation.
- A ticketing MCP server to create or update issues.

The AI application does not need a completely different integration pattern for each service. MCP provides a common protocol for communicating with the connected servers.

## How MCP works

MCP uses a host-client-server architecture. The three main components are:

### Host

The host is the AI application that the user interacts with.

An AI-powered IDE, desktop assistant, or other LLM application can act as a host.

The host manages MCP connections and controls which servers the application can access.

### Client

The client is the connector between the host and an individual MCP server.

A host can create multiple clients, with each client maintaining a connection to a particular server. The client handles communication with that server and keeps the server connection isolated from other servers.

### Server

The server exposes capabilities to the client.

Those capabilities can include tools, resources, and prompts.

An MCP server does not need to contain an LLM. It can be a small program whose job is to expose a specific set of data or functions through MCP.

This separation is important.

The model does not directly connect to every database, API, or local process. The host manages the connections, while MCP servers expose focused capabilities.

## MCP's three core primitives

The simplest way to understand MCP is to separate resources, tools, and prompts.

### Resources

Resources provide contextual information to an AI application.

A resource could represent:

- A document
- A database record
- A file
- Repository information
- Application data

The important distinction is that a resource represents information that can be provided as context.

For example, an MCP server for a documentation system could expose:

```
docs://product/authentication

```
A client can retrieve that resource and make its contents available to the AI application.

### Tools
Tools are executable functions. A server could expose a tool called:
```
search_documents


```
The tool might accept:

```
{
  "query": "How does authentication work?"
}

```
The server then performs the search and returns the result.

Tools can interact with external systems, including APIs and databases. Because tools can cause actions or access data, applications should treat tool invocation as a security-sensitive operation and provide appropriate user control.

### Prompts

Prompts are reusable templates exposed by an MCP server. For example, a documentation server might provide:
```
review_documentation


```

with an argument such as:


```
language = TypeScript

```
The client can retrieve the prompt and use it as part of an interaction with the model.

Prompts are intended to be user-controlled, while tools are generally model-controlled and resources are application-controlled.

## MCP vs. an API

MCP does not replace APIs. An API defines how software communicates with a particular service.

For example, a weather service might expose:

```
GET /forecast

```
An application can call that endpoint directly.

MCP provides a standardized interface through which an AI application can discover and use capabilities such as that weather lookup.

The distinction becomes clearer when you think about the layers involved:

```
AI application
      |
    MCP
      |
MCP server
      |
     API
      |
External service


```
The MCP server can act as the bridge between the AI application and the underlying service.

This means a company does not necessarily need to replace its existing APIs to support MCP. It can build an MCP server that exposes useful operations through the existing systems.

## MCP vs. RAG
MCP and retrieval-augmented generation solve different problems.

RAG is primarily a method for giving an LLM relevant information before it generates an answer.

A typical RAG pipeline looks like this:


```
Documents
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector store
    ↓
Retrieval
    ↓
Relevant context
    ↓
LLM
    ↓
Answer


```
MCP is a protocol for connecting an AI application to external context and capabilities. For example:

```
AI application
      ↓
MCP client
      ↓
MCP server
      ↓
Search tool
      ↓
Knowledge base


```
The two can also work together. An MCP server could expose a search tool backed by a vector database. The AI application could then use MCP to access the retrieval system.

So MCP is not an alternative to RAG. MCP can provide a standardized way for an AI application to access a RAG system.

## Build a simple MCP server

The easiest way to understand MCP is to build a small server.

The current TypeScript SDK v2 uses the @modelcontextprotocol/server package. The official quickstart requires Node.js 20 or later and uses tsx to run TypeScript directly.

Create a project:

```
mkdir weather-mcp
cd weather-mcp
npm init -y
npm pkg set type=module
npm install @modelcontextprotocol/server zod tsx
mkdir src

```
The type=module setting is required because the current SDK uses ES modules.
Create:

```
src/index.ts

```
Then import the server and stdio transport:

```
import { McpServer } from "@modelcontextprotocol/server";
import { serveStdio } from "@modelcontextprotocol/server/stdio";
import * as z from "zod/v4";

```
Create the MCP server:

```
function createServer() {
  const server = new McpServer({
    name: "weather",
    version: "1.0.0",
  });

```
  return server;

  ```
}

```
Now register a tool.

For a simple example, imagine a tool that receives a city and returns a message:

```
function createServer() {
  const server = new McpServer({
    name: "weather",
    version: "1.0.0",
  });

  server.registerTool(
    "get_weather",
    {
      description: "Get the current weather for a city",
      inputSchema: {
        city: z.string(),
      },
    },
    async ({ city }) => {
      return {
        content: [
          {
            type: "text",
            text: `Weather requested for ${city}`,
          },
        ],
      };
    }
  );

  return server;
}


```
Finally, start the server over stdio:

```
void serveStdio(createServer);
console.error("weather MCP server running on stdio");

```
The complete server now looks like this:

```
import { McpServer } from "@modelcontextprotocol/server";
import { serveStdio } from "@modelcontextprotocol/server/stdio";
import * as z from "zod/v4";

function createServer() {
  const server = new McpServer({
    name: "weather",
    version: "1.0.0",
  });

  server.registerTool(
    "get_weather",
    {
      description: "Get the current weather for a city",
      inputSchema: {
        city: z.string(),
      },
    },
    async ({ city }) => {
      return {
        content: [
          {
            type: "text",
            text: `Weather requested for ${city}`,
          },
        ],
      };
    }
  );

  return server;
}

void serveStdio(createServer);

console.error("weather MCP server running on stdio");


```
The official SDK example uses the same basic pattern: create an McpServer, register a tool, and pass the server factory to serveStdio.

## Run the server

Start it with:

```
npx tsx src/index.ts

```

You should see:

```
weather MCP server running on stdio


```
The process is now waiting for an MCP client. There is an important detail here.

With stdio transport, stdout is used for protocol communication. Do not write ordinary logs to stdout because they can interfere with the JSON-RPC messages. The official SDK guide recommends writing logs to stderr instead.

That is why the example uses:

```
console.error("weather MCP server running on stdio");


```
instead of:

```
console.log("weather MCP server running on stdio");


```
## Test the server with MCP Inspector

You can test an MCP server without connecting it to a full AI application.

The MCP Inspector is a development tool that can launch an MCP server and provide an interface for interacting with it.

Run:

```
npx @modelcontextprotocol/inspector npx tsx src/index.ts


```
The Inspector can connect to the local server over stdio and expose its available tools for testing.

For our example, the server should expose:

```
get_weather

with one input:
city

You can call the tool with:
{
  "city": "Lagos"
}


```
The server should return:

```
Weather requested for Lagos


```
This is a small example, but the underlying pattern is the same when the tool calls a real API or database.

## What happens when a model uses an MCP tool?

Consider an AI assistant receiving this request:

What is the weather in Lagos?

The application can make the MCP tool available to the model.

The model determines that the get_weather tool is relevant and supplies the required argument:

```
{
  "city": "Lagos"
}


```
The MCP client sends the tool call to the server.

The server executes the function and returns the result.

The application then provides that result to the model so it can produce the final response.

Conceptually:

```
User
  ↓
AI application
  ↓
LLM
  ↓
MCP tool selection
  ↓
MCP client
  ↓
MCP server
  ↓
External API
  ↓
MCP server
  ↓
MCP client
  ↓
LLM
  ↓
Final response

```
The important part is that the model does not need to know how the underlying weather API works.

It only needs to understand the tool's name, description, input schema, and returned result.

## Security considerations

MCP servers can expose data and executable functions, so security needs to be part of the integration rather than an afterthought.

A tool might only read information, but another tool could create a database record, modify a file, send an email, or trigger an external API.

The MCP specification recommends user consent and control over data access and tool operations. It also states that tool descriptions and annotations should not automatically be treated as trusted instructions.

A practical MCP integration should therefore answer questions such as:

- What data can this server access?
- Which tools can the model invoke?
- Which operations require user confirmation?
- What permissions does the server have?
- What happens if a tool receives unexpected input?
- What information is sent to the external service?

An MCP connection should expose only the capabilities that the AI application actually needs.

## Local vs. remote MCP servers

MCP servers can run locally or remotely.

A local server can run as a process on the same machine as the AI application. The stdio example above uses this model.

A remote MCP server can run as a service that an application connects to over the network.

The current MCP specification has moved toward a stateless protocol core for HTTP-based deployments. The 2026-07-28 specification removed the previous handshake and session requirements from the core and added request metadata that allows requests to be routed across ordinary HTTP infrastructure.

This matters when MCP servers need to operate at production scale.

A remote server should not be designed around assumptions that every request must reach the same server instance. The current specification is designed to support routing requests across instances behind standard load-balancing infrastructure.

## When should you use MCP?

MCP is useful when an AI application needs standardized access to external context or capabilities.

Good use cases include:
- Connecting an AI coding assistant to repositories.
- Giving an agent access to internal documentation.
- Connecting an AI application to business databases.
- Exposing search functionality to an agent.
- Allowing an AI assistant to interact with external APIs.
- Connecting multiple AI applications to the same integration.

MCP is less useful when there is no AI application involved.

If one backend service simply needs to call another backend service, a normal API may be the simpler solution.

The question is not whether MCP is newer or more capable than an API.

The question is whether an AI application needs a standardized way to discover and use external context or capabilities.

## Key takeaways

MCP gives AI applications a standard interface for connecting to external data and tools.

Its architecture separates the AI application, the MCP client, and the MCP server.

Its three core server primitives are resources, tools, and prompts.

MCP does not replace APIs or RAG. An MCP server can sit in front of existing APIs, databases, search systems, or retrieval pipelines.

For developers, the useful mental model is simple:

MCP is the interface.

The server provides the capability.

The client manages the connection.

The AI application decides when that capability is useful.


The most important part of working with MCP is not memorizing its terminology. It is understanding what capability you need to expose, what data that capability can access, and how the AI application should safely use it.


