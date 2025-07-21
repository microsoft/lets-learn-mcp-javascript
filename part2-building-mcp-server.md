#### **3. Building the Basic MCP Server**  

Now let's create our MCP server. Open `main.ts` and let's build it step by step.

**Step 1: Add Required Imports**  
Open `main.ts` and add:  
  ```ts
  import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
  import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
  import { z } from "zod";
  ```

**Step 2: Create the Server Instance**  
Add:  
  ```ts
  const server = new McpServer({
    name: "Weather Server",
    version: "1.0.0"
  });
  ```

The server manages all communication using the MCP protocol between clients (like VS Code) and your tools.

**Step 3: Define your first tool**  
Tools are functions that AI agents can call. Let's create a get-weather tool: 
  ```ts
  server.tool(
    'get-weather',
    'Tool to get the weather of a city',
    {
      city: z.string().describe("The name of the city to get the weather for")
    },
    async({ city }) => {
      return {
        content: [
          {
            type: "text",
            text: `The weather in ${city} is sunny`
          }
        ]
      };
    }
  );
  ```

**Breaking down the tool definition:**

1. **Tool ID**: 'get-weather' - Unique identifier
2. **Description**: Helps AI agents understand what this tool does
3. **Schema**: Defines parameters (city must be a string)
4. **Function**: The actual code that runs when called

**How it works:**

- AI agent sees: "Tool to get the weather of a city"
- AI agent calls it with: { city: "Paris" }
- Zod validates the input
- Function returns: "The weather in Paris is sunny"

**Step 4: Set Up Communication**  
Finally, we need to set up how our server communicates with AI clients: 
  ```ts
  const transport = new StdioServerTransport();
  server.connect(transport);
  ```

**What's happening here:**

StdioServerTransport uses your terminal's input/output for communication
  - Perfect for local development
  - The server reads requests from stdin and writes responses to stdout
  - MCP protocol handles all the message formatting automatically

```js
// Complete main.ts file
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { z } from "zod";

const server = new McpServer({
  name: "Weather Server",
  version: "1.0.0"
});

server.tool(
  'get-weather',
  'Tool to get the weather of a city',
  {
    city: z.string().describe("The name of the city to get the weather for")
  },
  async({ city }) => {
    return {
      content: [
        {
          type: "text",
          text: `The weather in ${city} is sunny`
        }
      ]
    };
  }
);

const transport = new StdioServerTransport();
server.connect(transport);
```
🎉 Congratulations! You've built your first MCP server. Let's test it works!


## Step 4: Testing with MCP Inspector

Before adding real weather data, let's test our server using the MCP Inspector, a web-based debugging tool for MCP servers.

**Launch the Inspector**  
Run this command to open the MCP Inspector for your server:

  ```bash
  npx -y @modelcontextprotocol/inspector npx -y tsx main.ts
  ```
After running the command, you'll see terminal output with:

- A localhost URL (like http://127.0.0.1:6274)
- A unique session token
- A direct link with the token pre-filled

_Tip: Click the link with the token already included to avoid manual entry._

### Connect and Test
1. **Connect**: Click the "Connect" button in the Inspector
2. **Navigate**: Click "Tools" in the top navigation
3. **Select**: Choose your get-weather tool
4. **Test**: Enter a city name (like "Palma de Mallorca") and click "Run Tool"
5. **View Response**: You should see the response: "The weather in Palma de Mallorca is sunny"

**Troubleshooting:**

**Connection Error?** Make sure you used the link with the pre-filled token

Perfect! Your MCP server is working. Now let's make it actually useful.

## Step 5: Adding Real Weather Data

Time to make our server actually useful! We'll integrate with [Open-Meteo](https://open-meteo.com/), a free weather API that requires no API key.

### How the Weather API Works

Before diving into the implementation, it's important to understand the two API calls we will use:

1. **Geocoding API**: Converts city names into geographic coordinates (latitude and longitude).
2. **Weather API**: Retrieves weather data using the geographic coordinates.

### Geocoding API
Use the [geocoding endpoint](https://open-meteo.com/en/docs/geocoding-api) to get the latitude and longitude of a city. Example URL:
  ```bash
  https://geocoding-api.open-meteo.com/v1/search?name=Berlin&count=10&language=en&format=json
  ```

Replace the static city name with `${city}` using template literals.

Then return our data as JSON and extract the latitude and longitude from the first result.

Include error handling to notify the AI agent if the city doesn't exist.
```ts
//...
async({ city }) => {
      try {
        // Fetch geocoding data
        const response = await fetch(
          `https://geocoding-api.open-meteo.com/v1/search?name=${city}&count=10&language=en&format=json`
        );
        const data = await response.json();

        const { latitude, longitude } = data.results[0];

        if (data.results.length === 0) { {
          return {
            content: [
              {
                type: "text",
                text: `Sorry, I couldn't find a city named "${city}".`
              }
            ]
          };
        }
      } catch (error) {
        return {
          content: [
            {
              type: "text",
              text: `Error fetching geocoding data: ${error.message}`
            }
          ]
        };
      }
    }
  );
  //...
  ```

#### Fetch the Weather Data
We can now call the [Weather Forecast API](https://open-meteo.com/en/docs). 

By passing in the latitude and longitude values, we can get the weather for the city and choose which data to return. 

Let's select the current weather and include - temperature
- apparent temperature
- humidity
- precipitation

Let's select just today's data. Then, down below, we can find our endpoint which we can copy into our code.


Back in VS Code, we will modify our `get-weather` tool to include the real weather data:

- Create a new fetch to our API and store it as `weatherResponse`. 
- To call the correct city we need to use the latitude and longitude we obtained earlier from our data. 

  ```ts
  const weatherResponse = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&hourly=temperature_2m&current=temperature_2m,relative_humidity_2m,wind_speed_10m,precipitation,rain,showers,cloud_cover,apparent_temperature`);


  ```
- Then we return our weather response as JSON.

  ```ts
  const weatherData = await weatherResponse.json();
  ```
- Finally we modify our static return text to include the actual weather data using `JSON.stringify`, which Converts a JavaScript value to a JavaScript Object Notation (JSON) string. We return the whole JSON object to the LLM and let it figure out what to do with the data.

  ```ts
 
  return {
    content: [
      {
        type: "text",
        text: JSON.stringify(weatherData, null, 2),
      }
    ]
  };
  ```

```js
// Complete main.ts file with real weather data integration
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { z } from "zod";

const server = new McpServer({
  name: "Weather Server",
  version: "1.0.0"
});

server.tool(
  'get-weather',
  'Tool to get the weather of a city',
  {
    city: z.string().describe("The name of the city to get the weather for")
  },
  async({ city }) => {
    try {
      const response = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${city}&count=10&language=en&format=json`);
      const data = await response.json();
      
      if (data.results.length === 0) {
        return {
          content: [
            {
              type: "text",
              text: `No results found for city: ${city}`
            }
          ]
        };
      }
      
      const { latitude, longitude } = data.results[0];
      const weatherResponse = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&hourly=temperature_2m,precipitation,apparent_temperature,relative_humidity_2m&forecast_days=1`);
      
      const weatherData = await weatherResponse.json();
      
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(weatherData, null, 2)
          }
        ]
      };
    } catch (error) {
      return {
        content: [
          {
            type: "text",
            text: `Error fetching weather data: ${error.message}`
          }
        ]
      };
    }
  }
);

const transport = new StdioServerTransport();
server.connect(transport);

```

**Test your real data works:**  
- After implementing the `get-weather` tool, restart the server and test it using the MCP Inspector. Enter a city name like "Berlin" and verify that the response includes the current temperature.
