# Building Your First MCP Weather Server

## Step 1: Setting Up the Project

**1. Initialize the Project**  
Create a new directory and initialize it with npm: 
  ```bash
  mkdir mcp-weather-server
  cd mcp-weather-server
  npm init -y
  ```
**2. Create the Main File**  
Create our main TypeScript file (This is the only file we will need for this server, yes only one file!):

Run:  
  ```bash
  touch main.ts
  ```

**3. Configure `package.json`**  
Open the project in VS Code (or your preferred editor) and modify `package.json` to enable ES modules by adding the `type: module`:
 
  ```json
  {
    "name": "mcp-weather-server",
    "version": "1.0.0",
    "type": "module",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    }
  }
  ```
Why ES modules? The MCP SDK uses modern JavaScript modules, so we need to enable them in our project.

## Step 2: Install Dependencies

Our MCP server needs two key libraries:

**1. Install the MCP SDK and Zod:**  
The Model Context Protocol SDK provides everything needed to build MCP servers:

  ```bash
  npm install @modelcontextprotocol/sdk
  ```
**2. Install Zod for data validation:** 
Zod ensures our server receives valid data from AI agents: 
  ```bash
  npm install zod
  ```

Your package.json dependencies should now look like this: 
  ```json
  {
    "dependencies": {
      "@modelcontextprotocol/sdk": "^1.13.1",
      "zod": "^3.25.67"
    }
  }
  ```






