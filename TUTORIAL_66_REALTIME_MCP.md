# Tutorial 66: Real-time MCP Patterns

> **Advanced MCP patterns including dynamic tool updates, list_changed notifications, and live data integration**

## Table of Contents

1. [Overview](#overview)
2. [Dynamic Tool Registration](#dynamic-tool-registration)
3. [list_changed Notifications](#list_changed-notifications)
4. [Real-time Data Patterns](#real-time-data-patterns)
5. [Long-Running Operations](#long-running-operations)
6. [Event-Driven MCP Servers](#event-driven-mcp-servers)
7. [State Management](#state-management)
8. [Performance Optimization](#performance-optimization)
9. [Error Handling](#error-handling)
10. [Quick Reference](#quick-reference)

---

## Overview

Model Context Protocol (MCP) supports dynamic capabilities beyond static tool definitions. This tutorial covers advanced patterns for building responsive, real-time MCP integrations.

### Key Concepts

| Concept | Description |
|---------|-------------|
| Dynamic tools | Tools that change during runtime |
| list_changed | Notification when tool list updates |
| Streaming | Progressive results for long operations |
| State | Maintaining context across calls |

### When to Use Real-time Patterns

- Database schema that changes dynamically
- API endpoints discovered at runtime
- Feature flags controlling tool availability
- Context-dependent tool sets
- Long-running queries with progress updates

---

## Dynamic Tool Registration

### Basic Dynamic Tool Server

```typescript
// dynamic-tools-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";

class DynamicToolServer {
  private tools: Map<string, ToolDefinition> = new Map();
  private server: Server;

  constructor() {
    this.server = new Server(
      { name: "dynamic-tools", version: "1.0.0" },
      { capabilities: { tools: { listChanged: true } } }
    );

    this.setupHandlers();
  }

  private setupHandlers() {
    this.server.setRequestHandler("tools/list", async () => {
      return {
        tools: Array.from(this.tools.values())
      };
    });

    this.server.setRequestHandler("tools/call", async (request) => {
      const tool = this.tools.get(request.params.name);
      if (!tool) {
        throw new Error(`Unknown tool: ${request.params.name}`);
      }
      return await tool.handler(request.params.arguments);
    });
  }

  // Add tool dynamically
  async registerTool(name: string, definition: ToolDefinition) {
    this.tools.set(name, definition);
    await this.notifyToolsChanged();
  }

  // Remove tool dynamically
  async unregisterTool(name: string) {
    this.tools.delete(name);
    await this.notifyToolsChanged();
  }

  private async notifyToolsChanged() {
    await this.server.sendNotification("notifications/tools/list_changed", {});
  }
}
```

### Use Case: Database Schema Tools

```typescript
class DatabaseSchemaServer {
  private tables: Map<string, TableSchema> = new Map();

  async discoverSchema() {
    // Fetch current database schema
    const result = await db.query(
      "SELECT table_name, column_name, data_type FROM information_schema.columns"
    );

    // Clear existing tools
    this.tools.clear();

    // Create tools for each table
    for (const table of groupByTable(result)) {
      this.registerTool(`query_${table.name}`, {
        name: `query_${table.name}`,
        description: `Query the ${table.name} table`,
        inputSchema: this.buildQuerySchema(table)
      });

      this.registerTool(`insert_${table.name}`, {
        name: `insert_${table.name}`,
        description: `Insert into ${table.name} table`,
        inputSchema: this.buildInsertSchema(table)
      });
    }

    await this.notifyToolsChanged();
  }

  // Refresh schema periodically
  startSchemaWatcher(intervalMs: number = 60000) {
    setInterval(() => this.discoverSchema(), intervalMs);
  }
}
```

---

## list_changed Notifications

### Server Capability Declaration

```typescript
const server = new Server(
  { name: "my-server", version: "1.0.0" },
  {
    capabilities: {
      tools: {
        listChanged: true  // Enable notifications
      }
    }
  }
);
```

### Sending Notifications

```typescript
// Notify Claude that tool list has changed
await server.sendNotification("notifications/tools/list_changed", {});
```

### When to Send list_changed

| Trigger | Example |
|---------|---------|
| Tool added | New API endpoint discovered |
| Tool removed | Feature disabled |
| Tool modified | Schema changed |
| Permissions changed | User role updated |

### Client Handling (Claude Code)

When Claude Code receives `list_changed`:
1. Re-fetches tool list from server
2. Updates internal tool registry
3. Makes new tools available immediately

---

## Real-time Data Patterns

### Pattern 1: Live Data Tools

```typescript
// Stock price server with real-time updates
class StockServer {
  private subscriptions: Map<string, WebSocket> = new Map();

  async handleCall(request: CallToolRequest) {
    const { symbol } = request.params.arguments;

    switch (request.params.name) {
      case "get_stock_price":
        return await this.getPrice(symbol);

      case "subscribe_price":
        return this.subscribePrice(symbol);

      case "get_price_history":
        return await this.getHistory(symbol, request.params.arguments.period);
    }
  }

  private async getPrice(symbol: string) {
    const price = await this.fetchCurrentPrice(symbol);
    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          symbol,
          price: price.current,
          change: price.change,
          timestamp: new Date().toISOString()
        })
      }]
    };
  }
}
```

### Pattern 2: Database Query with Progress

```typescript
async handleLargeQuery(request: CallToolRequest) {
  const { query, estimatedRows } = request.params.arguments;

  // Start query
  const cursor = await db.query(query).cursor();

  let processed = 0;
  const results = [];

  for await (const row of cursor) {
    results.push(row);
    processed++;

    // Report progress periodically
    if (processed % 1000 === 0) {
      await this.reportProgress({
        processed,
        estimated: estimatedRows,
        percentage: Math.round((processed / estimatedRows) * 100)
      });
    }
  }

  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        rows: results.length,
        data: results.slice(0, 100),  // First 100 rows
        hasMore: results.length > 100
      })
    }]
  };
}
```

---

## Long-Running Operations

### Progress Reporting Pattern

```typescript
interface ProgressUpdate {
  stage: string;
  progress: number;  // 0-100
  message: string;
  estimatedTimeRemaining?: number;
}

class LongRunningServer {
  private operations: Map<string, Operation> = new Map();

  async startOperation(request: CallToolRequest) {
    const operationId = crypto.randomUUID();

    // Start background operation
    const operation = this.runInBackground(
      request.params.arguments,
      (progress) => this.updateProgress(operationId, progress)
    );

    this.operations.set(operationId, operation);

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          operationId,
          status: "started",
          message: "Operation started. Use check_operation to monitor progress."
        })
      }]
    };
  }

  async checkOperation(operationId: string) {
    const operation = this.operations.get(operationId);

    if (!operation) {
      return { status: "not_found" };
    }

    return {
      status: operation.status,
      progress: operation.progress,
      result: operation.result,
      error: operation.error
    };
  }
}
```

### Timeout Handling

```typescript
async executeWithTimeout(
  operation: () => Promise<any>,
  timeoutMs: number = 30000
) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await operation();
  } finally {
    clearTimeout(timeout);
  }
}
```

---

## Event-Driven MCP Servers

### WebSocket Integration

```typescript
class EventDrivenServer {
  private eventQueue: EventEmitter = new EventEmitter();

  constructor() {
    // Connect to external event source
    const ws = new WebSocket("wss://events.example.com");

    ws.on("message", (data) => {
      const event = JSON.parse(data);
      this.handleExternalEvent(event);
    });
  }

  private handleExternalEvent(event: ExternalEvent) {
    switch (event.type) {
      case "schema_change":
        this.refreshTools();
        break;

      case "new_data":
        this.eventQueue.emit("data", event.payload);
        break;

      case "alert":
        this.eventQueue.emit("alert", event.payload);
        break;
    }
  }

  async waitForEvent(eventType: string, timeoutMs: number = 30000) {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        reject(new Error("Timeout waiting for event"));
      }, timeoutMs);

      this.eventQueue.once(eventType, (data) => {
        clearTimeout(timeout);
        resolve(data);
      });
    });
  }
}
```

### Polling Pattern

```typescript
class PollingServer {
  private cache: Map<string, CachedResult> = new Map();
  private pollingIntervals: Map<string, NodeJS.Timeout> = new Map();

  startPolling(resource: string, intervalMs: number) {
    const poll = async () => {
      try {
        const data = await this.fetchResource(resource);
        const cached = this.cache.get(resource);

        if (JSON.stringify(data) !== JSON.stringify(cached?.data)) {
          this.cache.set(resource, {
            data,
            timestamp: Date.now()
          });

          // Notify about data change
          await this.notifyDataChanged(resource);
        }
      } catch (error) {
        console.error(`Polling error for ${resource}:`, error);
      }
    };

    // Initial fetch
    poll();

    // Set up interval
    const interval = setInterval(poll, intervalMs);
    this.pollingIntervals.set(resource, interval);
  }

  stopPolling(resource: string) {
    const interval = this.pollingIntervals.get(resource);
    if (interval) {
      clearInterval(interval);
      this.pollingIntervals.delete(resource);
    }
  }
}
```

---

## State Management

### Session State

```typescript
class StatefulServer {
  private sessions: Map<string, SessionState> = new Map();

  async handleCall(request: CallToolRequest) {
    const sessionId = request.params._meta?.sessionId || "default";
    let session = this.sessions.get(sessionId);

    if (!session) {
      session = this.createSession(sessionId);
      this.sessions.set(sessionId, session);
    }

    // Update session context
    session.lastAccess = Date.now();

    switch (request.params.name) {
      case "set_context":
        return this.setContext(session, request.params.arguments);

      case "get_context":
        return this.getContext(session);

      case "clear_context":
        return this.clearContext(session);

      default:
        return this.executeWithContext(session, request);
    }
  }

  private createSession(id: string): SessionState {
    return {
      id,
      created: Date.now(),
      lastAccess: Date.now(),
      context: {},
      history: []
    };
  }

  // Clean up old sessions
  startSessionCleanup(maxAgeMs: number = 3600000) {
    setInterval(() => {
      const now = Date.now();
      for (const [id, session] of this.sessions) {
        if (now - session.lastAccess > maxAgeMs) {
          this.sessions.delete(id);
        }
      }
    }, 60000);  // Check every minute
  }
}
```

### Transaction State

```typescript
class TransactionServer {
  private transactions: Map<string, Transaction> = new Map();

  async beginTransaction() {
    const txId = crypto.randomUUID();
    const tx = await this.db.beginTransaction();

    this.transactions.set(txId, {
      id: txId,
      connection: tx,
      operations: [],
      started: Date.now()
    });

    return { transactionId: txId };
  }

  async executeInTransaction(txId: string, operation: Operation) {
    const tx = this.transactions.get(txId);
    if (!tx) {
      throw new Error("Transaction not found");
    }

    const result = await tx.connection.execute(operation);
    tx.operations.push({ operation, result, timestamp: Date.now() });

    return result;
  }

  async commitTransaction(txId: string) {
    const tx = this.transactions.get(txId);
    if (!tx) {
      throw new Error("Transaction not found");
    }

    await tx.connection.commit();
    this.transactions.delete(txId);

    return {
      committed: true,
      operations: tx.operations.length
    };
  }

  async rollbackTransaction(txId: string) {
    const tx = this.transactions.get(txId);
    if (!tx) {
      throw new Error("Transaction not found");
    }

    await tx.connection.rollback();
    this.transactions.delete(txId);

    return { rolledBack: true };
  }
}
```

---

## Performance Optimization

### Caching Strategy

```typescript
class CachedMCPServer {
  private cache: LRUCache<string, CachedResult>;

  constructor() {
    this.cache = new LRUCache({
      max: 1000,
      ttl: 1000 * 60 * 5  // 5 minutes
    });
  }

  async handleCall(request: CallToolRequest) {
    const cacheKey = this.getCacheKey(request);

    // Check cache
    const cached = this.cache.get(cacheKey);
    if (cached && !this.isStale(cached)) {
      return {
        ...cached.result,
        _meta: { cached: true, age: Date.now() - cached.timestamp }
      };
    }

    // Execute and cache
    const result = await this.execute(request);
    this.cache.set(cacheKey, {
      result,
      timestamp: Date.now()
    });

    return result;
  }

  private getCacheKey(request: CallToolRequest): string {
    return JSON.stringify({
      name: request.params.name,
      arguments: request.params.arguments
    });
  }
}
```

### Connection Pooling

```typescript
class PooledDatabaseServer {
  private pool: Pool;

  constructor() {
    this.pool = new Pool({
      host: process.env.DB_HOST,
      database: process.env.DB_NAME,
      max: 20,  // Maximum connections
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 2000
    });
  }

  async executeQuery(query: string, params: any[]) {
    const client = await this.pool.connect();
    try {
      return await client.query(query, params);
    } finally {
      client.release();
    }
  }
}
```

### Batch Operations

```typescript
async handleBatchRequest(requests: ToolRequest[]) {
  // Group by operation type
  const grouped = groupBy(requests, r => r.params.name);

  const results = [];

  for (const [name, group] of Object.entries(grouped)) {
    // Execute batch
    const batchResults = await this.executeBatch(name, group);
    results.push(...batchResults);
  }

  return results;
}
```

---

## Error Handling

### Graceful Degradation

```typescript
async handleCall(request: CallToolRequest) {
  try {
    return await this.primaryHandler(request);
  } catch (error) {
    // Log error
    console.error("Primary handler failed:", error);

    // Try fallback
    try {
      return await this.fallbackHandler(request);
    } catch (fallbackError) {
      // Return graceful error response
      return {
        content: [{
          type: "text",
          text: JSON.stringify({
            error: true,
            message: "Operation temporarily unavailable",
            suggestion: "Please try again later or use alternative approach"
          })
        }],
        isError: true
      };
    }
  }
}
```

### Retry with Backoff

```typescript
async executeWithRetry(
  operation: () => Promise<any>,
  maxRetries: number = 3
) {
  let lastError: Error;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;

      // Exponential backoff
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}
```

---

## Quick Reference

### Server Capabilities

```typescript
{
  capabilities: {
    tools: {
      listChanged: true  // Enable dynamic tools
    },
    prompts: {
      listChanged: true  // Enable dynamic prompts
    },
    resources: {
      subscribe: true,   // Enable subscriptions
      listChanged: true  // Enable dynamic resources
    }
  }
}
```

### Notification Types

| Notification | When to Use |
|--------------|-------------|
| `tools/list_changed` | Tools added/removed/modified |
| `resources/list_changed` | Resources changed |
| `prompts/list_changed` | Prompts changed |
| `resources/updated` | Resource content updated |

### Common Patterns

```typescript
// Dynamic tool registration
await server.sendNotification("notifications/tools/list_changed", {});

// Progress reporting
return {
  content: [{
    type: "text",
    text: JSON.stringify({ progress: 50, status: "processing" })
  }]
};

// Session state
const sessionId = request.params._meta?.sessionId;

// Error response
return {
  content: [{ type: "text", text: "Error message" }],
  isError: true
};
```

### Best Practices

1. **Always declare capabilities** in server initialization
2. **Send list_changed** when tools actually change
3. **Use caching** for expensive operations
4. **Implement timeouts** for long operations
5. **Clean up state** periodically
6. **Handle errors gracefully** with fallbacks
7. **Pool connections** to external resources

---

## Sources

- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [MCP SDK Documentation](https://github.com/modelcontextprotocol/typescript-sdk)
- [Claude Code MCP Integration](https://code.claude.com/docs/en/mcp.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
