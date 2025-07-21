# Connecting MCP Server to VS Code**  

**Step 1: Register the Server**  
#### Register the Server

1. Open the VS Code Command Palette (`Cmd/Ctrl + Shift + P`).
2. Type **"MCP: Add Server"**.
3. Choose **"Local server using stdio"**.
4. Enter the command: `npx -y tsx main.ts`.
5. Give it a name like `my-weather-server`.
6. Choose **local setup**.

This creates a `.vscode/mcp.json` file in your project:

```json
{
  "inputs": [],
  "servers": {
    "my-weather-server": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "tsx",
        "main.ts"
      ]
    }
  }
}
```

**Step 2: Test with GitHub Copilot**  
1. Start the server by clicking the "Start" button next to your server name in the MCP panel.
2. Switch to **Agent Mode** in the Copilot sidebar.
3. Ask: "What's the weather like in Tokyo?"

You should see a prompt in Copilot asking if it can run the `get-weather` tool with the input `{ city: "Tokyo" }`. Confirm the prompt to proceed.

**Expected Result:**
Instead of raw JSON, you should see a beautifully formatted weather report, such as:

```md
> **Weather in Tokyo Today**  
> **Temperature:** 28°C (feels like 32°C)  
> **Humidity:** 75%  
> **Conditions:** Partly cloudy with light rain expected in the evening
```

This demonstrates how seamlessly MCP servers integrate with AI tools like GitHub Copilot.

