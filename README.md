# MCP boilerplate: Model Context Protocol Server

This server implements the Model Context Protocol (MCP) for global use as a boilerplate. It provides a standardized way to connect AI models to different data sources and tools using the Model Context Protocol.

## Features

- Implements the MCP Server-Sent Events (SSE) transport
- Provides a robust structure for building custom MCP servers
- Includes example tools with proper type definitions
- Secure authentication with API key
- Logging capabilities with different severity levels
- Session management for multiple client connections
- Graceful shutdown handling for SIGINT and SIGTERM signals

## Tools

The server currently includes the following example tool:

- `calculator`: Performs basic arithmetic operations (add, subtract, multiply, divide)

For information on how to add your own custom tools, check out the [Extending the Boilerplate section](#extending-the-boilerplate).

## Configuration

The server configuration is centralized in `src/config.ts`. This makes it easy to adjust settings without modifying multiple files.

```typescript
// Essential configuration options
export const config = {
  server: {
    name: "mcp-boilerplate",
    version: "1.0.0",
    port: parseInt(process.env.PORT || "4005"),
    host: process.env.HOST || "localhost",
    apiKey: process.env.API_KEY || "dev_key",
  },
  sse: {
    // How often to send keepalive messages (in milliseconds)
    keepaliveInterval: 30000,
    // Whether to send ping events in addition to comments
    usePingEvents: true,
    // Initial connection message
    sendConnectedEvent: true,
  },
  tools: {
    // Number of retries for failed tool executions
    maxRetries: 3,
    // Delay between retries (in milliseconds)
    retryDelay: 1000,
    // Whether to send notifications about tool execution status
    sendNotifications: true,
  },
  logging: {
    // Default log level
    defaultLevel: "debug",
    // How often to send log messages (in milliseconds)
    logMessageInterval: 10000,
  },
};
```

### Troubleshooting SSE Timeouts

If you're experiencing "Body timeout error" with your MCP connection:

1. Decrease `keepaliveInterval` to send more frequent keepalive messages (e.g., 15000ms)
2. Ensure `usePingEvents` is enabled for additional connection stability
3. Check for any proxy timeouts if you're using a proxy server

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create a `.env` file with the following variables:

```
PORT=4005
API_KEY=your_api_key
```

3. Build the project:

```bash
npm run build
```

4. Start the server:

```bash
npm run start:sse
```

## Development

```bash
# Start in development mode with hot reloading
npm run start

# Start with PM2 for production
npm run start:pm2

# Development mode with nodemon
npm run dev
```

## API Endpoints

- `/health`: Health check endpoint that returns server status and version
- `/sse`: SSE endpoint for establishing MCP connections (requires API key)
- `/messages`: Message handling endpoint for client-server communication

## MCP Configuration

To connect to this MCP server from different clients, use the appropriate configuration below:

### Cursor, Windsurf, and other SSE-supporting clients

```json
{
  "mcpServers": {
    "mcp-server": {
      "url": "http://localhost:4005/sse?API_KEY={{your_api_key_here}}"
    }
  }
}
```

### Claude Desktop

```json
{
  "mcpServers": {
    "mcp-server": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://localhost:4005/sse?API_KEY={{your_api_key_here}}"
      ]
    }
  }
}
```

## Extending the boilerplate

### Adding Custom Tools

Follow these steps to add a new tool to the MCP server:

1. **Create your tool handler**:

   - Add your new tool handler in `src/tools.ts` file or create a new file in the `src/tools` directory
   - The tool should follow the `ToolHandler` interface

2. **Configure your tool**:

   - Add your tool configuration to the `toolConfigs` array in `src/tools.ts`
   - Define the name, description, input schema, and handler for your tool

3. **Export and register your tool**:
   - If you created a separate file, export your handler and import it in `src/tools.ts`
   - Make sure your tool is properly registered in the `toolConfigs` array

Example:

```typescript
// In src/tools.ts (adding directly to the toolConfigs array)
{
  name: "myTool",
  description: "My tool description",
  inputSchema: {
    type: "object" as const,
    properties: {},
    required: [],
  },
  handler: async () => {
    return createSuccessResult({ result: "Tool result" });
  },
}
```

## Error Handling

The server implements comprehensive error handling:

- All operations are wrapped in try/catch blocks
- Proper validation for parameters and inputs
- Appropriate error messages for better debugging
- Helper functions for creating standardized error and success responses

## Security Considerations

- API key authentication for all connections
- Type validation for all parameters
- No hard-coded sensitive information
- Proper error handling to prevent information leakage
- Session-based transport management

## MCP Protocol Features

This boilerplate supports the core MCP features:

- Tools: List and call tools with proper parameter validation
- Logging: Various severity levels (debug, info, notice, warning, error, critical, alert, emergency)
- Server configuration: Name, version, and capabilities

## Session Management

The server manages client sessions through:

- Unique session IDs for each client connection
- Tracking of active transports by session ID
- Automatic cleanup of disconnected sessions
- Connection status tracking

## Additional Resources

- [MCP Documentation](https://modelcontextprotocol.io/introduction)
- [MCP TypeScript SDK](https://modelcontextprotocol.io/typescript/index.html)

## License

This project is licensed under the [MIT License](LICENSE) - see the [LICENSE](LICENSE) file for details.
