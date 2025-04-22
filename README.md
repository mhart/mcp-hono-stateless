# Stateless Hono MCP Server

An example [Hono](https://hono.dev/) MCP server using Streamable HTTP, based off [the official Express example](https://github.com/modelcontextprotocol/typescript-sdk/blob/4d6197ac07776ab95a2d63a781514a75740cf746/src/examples/server/simpleStatelessStreamableHttp.ts),
deployable to [Cloudflare Workers](https://developers.cloudflare.com/workers/) (and anywhere else Hono runs).

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
+   const body = await c.req.json();
+   const { req, res } = toReqRes(c.req.raw);

    const server = getServer();
    try {
      const transport: StreamableHTTPServerTransport = new StreamableHTTPServerTransport({
        sessionIdGenerator: undefined,
      });
      await server.connect(transport);
-     await transport.handleRequest(req, res, req.body);
+     await transport.handleRequest(req, res, body);
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
