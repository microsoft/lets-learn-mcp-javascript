# MCP Weather Server

A simple Model Context Protocol (MCP) server that provides real-time weather data to AI agents like GitHub Copilot.

<a href="https://glama.ai/mcp/servers/@microsoft/lets-learn-mcp-javascript">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@microsoft/lets-learn-mcp-javascript/badge" alt="Weather Server MCP server" />
</a>

## Quick Start

Clone the repository:

```bash
git clone https://github.com/microsoft/lets-learn-mcp-javascript.git
cd mcp-weather-server
```

### 1. Install Dependencies

```bash
npm install
```

### 2. Run the Server

Test with MCP Inspector:

```bash
npx -y @modelcontextprotocol/inspector npx -y tsx main.ts
```

### 3. Use with VS Code

1. Open the `mcp.json`file in `.vscode` folder
2. Click the start server button above line 4
3. Open Chat mode and select agent and choose a modal that supports MCPs such as Claude Sonnet
4. Type or speak into the chat and ask it what the weather is like in your city

## Features

- 🌤️ Real-time weather data for any city
- 🌍 No API key required (uses Open-Meteo)
- 🤖 Works with GitHub Copilot and other MCP-compatible AI tools
- ⚡ Easy to test with MCP Inspector

## Usage Examples

Ask GitHub Copilot:
- "What's the weather like in Tokyo?"
- "How's the weather in London today?"
- "Give me the current weather for Paris"

## How It Works

The server provides a `get-weather` tool that:
1. Converts city names to coordinates using geocoding
2. Fetches current weather data from Open-Meteo API
3. Returns structured data that AI agents can format beautifully

## Code Structure

```typescript
// Creates MCP server with weather tool
const server = new McpServer({
  name: "Weather Server",
  version: "1.0.0"
});

// Defines the get-weather tool
server.tool('get-weather', 'Tool to get the weather of a city', ...);

// Connects via stdio transport
const transport = new StdioServerTransport();
server.connect(transport);
```

## Dependencies

- `@modelcontextprotocol/sdk` - MCP server framework
- `zod` - Schema validation
- `tsx` - TypeScript execution (for development)

## API Used

- [Open-Meteo](https://open-meteo.com/) - Free weather API with no authentication required