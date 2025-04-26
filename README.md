# Stateless Hono MCP Server

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/mhart/mcp-hono-stateless)

An example [Hono](https://hono.dev/) MCP server using Streamable HTTP, based off [the official Express example](https://github.com/modelcontextprotocol/typescript-sdk/blob/4d6197ac07776ab95a2d63a781514a75740cf746/src/examples/server/simpleStatelessStreamableHttp.ts),
using [fetch-to-node](https://github.com/mhart/fetch-to-node) to convert, deployable to [Cloudflare Workers](https://developers.cloudflare.com/workers/) (and anywhere else Hono runs).

The only real changes to the Express example are:

```diff
- import express, { Request, Response } from 'express';
+ import { Hono } from 'hono';
+ import { toFetchResponse, toReqRes } from 'fetch-to-node';

// ...

- const app = express();
- app.use(express.json());
+ const app = new Hono();

- app.post('/mcp', async (req: Request, res: Response) => {
+ app.post('/mcp', async (c) => {
+   const { req, res } = toReqRes(c.req.raw);

    const server = getServer();
    try {
      const transport: StreamableHTTPServerTransport = new StreamableHTTPServerTransport({
        sessionIdGenerator: undefined,
      });
      await server.connect(transport);
-     await transport.handleRequest(req, res, req.body);
+     await transport.handleRequest(req, res, await c.req.json());
      res.on('close', () => {
        console.log('Request closed');
        transport.close();
        server.close();
      });
+     return toFetchResponse(res);
    } catch (error) {
```

## Testing with an example MCP client

In one terminal:

```sh
npm start
```

In another:

```sh
node node_modules/@modelcontextprotocol/sdk/dist/esm/examples/client/simpleStreamableHttp.js
```

This will try to connect to the MCP server running on port 3000. You can use `connect <url>/mcp` to connect to a different host or port.

Then you can run commands like `list-prompts` or `list-tools` to verify your MCP server is working.

## Deploying

```sh
npm run deploy
```

Here's an example session against a Hono MCP server deployed on Cloudflare Workers:

```console
> connect https://mcp-hono-stateless.michael.workers.dev/mcp
Connecting to https://mcp-hono-stateless.michael.workers.dev/mcp...
Transport created with session ID: undefined
Connected to MCP server

> list-tools
Available tools:
  - start-notification-stream: Starts sending periodic notifications for testing resumability

> list-prompts
Available prompts:
  - greeting-template: A simple greeting prompt template

> call-tool start-notification-stream
Calling tool 'start-notification-stream' with args: {}

Notification #1: info - Periodic notification #1 at 2025-04-22T16:20:50.178Z
>
Notification #2: info - Periodic notification #2 at 2025-04-22T16:20:50.278Z
>
Notification #3: info - Periodic notification #3 at 2025-04-22T16:20:50.378Z
>
Notification #4: info - Periodic notification #4 at 2025-04-22T16:20:50.478Z
>
Notification #5: info - Periodic notification #5 at 2025-04-22T16:20:50.578Z
>
Notification #6: info - Periodic notification #6 at 2025-04-22T16:20:50.678Z
>
Notification #7: info - Periodic notification #7 at 2025-04-22T16:20:50.778Z
>
Notification #8: info - Periodic notification #8 at 2025-04-22T16:20:50.878Z
>
Notification #9: info - Periodic notification #9 at 2025-04-22T16:20:50.978Z
>
Notification #10: info - Periodic notification #10 at 2025-04-22T16:20:51.078Z
> Tool result:
  Started sending periodic notifications every 100ms
```
