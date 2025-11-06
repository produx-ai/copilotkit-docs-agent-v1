# CRUSH.md

## Project Overview

This is the CopilotKit monorepo - a framework for building AI-powered copilots and agentic applications. It includes the main CopilotKit package, SDKs, examples, and documentation.

## Essential Commands

### Development
- `pnpm install` - Install dependencies (workspace setup)
- `cd CopilotKit && pnpm dev` - Start development environment
- `cd docs && pnpm dev` - Start docs development server
- `pnpm build` - Build all packages
- `pnpm test` - Run test suites

### Testing
- `cd CopilotKit && pnpm test --filter=@copilotkit/runtime` - Test specific package
- `cd examples/e2e && pnpm test` - Run e2e tests with Playwright

### Deployment/Release
- `cd CopilotKit && pnpm release` - Release packages (internal script)
- `cd docs && pnpm build` - Build documentation

## Code Organization

### Core Packages (`CopilotKit/packages/`)
- `react-core/` - Main React hooks and context providers
- `react-ui/` - Pre-built UI components (CopilotChat, CopilotSidebar, etc.)
- `react-textarea/` - AI-enhanced textarea component
- `runtime/` - Server-side runtime for action execution and agent processing
- `shared/` - Common types and utilities shared across packages

### SDKs
- `sdk-python/` - Python SDK for agent development (crewai, langgraph integrations)
- `CopilotKit/packages/sdk-js/` - JavaScript SDK components

### Documentation (`docs/content/docs/`)
- Framework-specific guides (langgraph/, crewai-crews/, direct-to-llm/, etc.)
- Reference documentation for hooks, components, and classes
- Tutorials and examples

### Examples (`examples/`)
Organized by framework/agent type:
- Multi-agent systems: `coagents-*` folders
- Framework integrations: `crewai-crews/`, `langgraph/`, `mastra/`, etc.
- UI-focused: `copilot-*` folders

## Key Patterns

### Runtime Configuration
```typescript
const runtime = new CopilotRuntime({
  actions: ({ properties, url }) => [
    {
      name: "myAction",
      description: "Action description",
      parameters: [{ name: "param", type: "string", required: true }],
      handler: async (params) => {
        // Implementation
        return result;
      }
    }
  ]
});
```

### Action Definition Pattern
Actions use consistent structure:
- `name`: kebab-case identifier
- `description`: Clear, actionable description
- `parameters`: Array of Parameter objects with name, type, description, required
- `handler`: Async function taking destructured parameters

### Component Patterns
All CopilotKit components follow React patterns:
- Use hooks like `useCopilotChat`, `useCopilotAction`, `useCopilotReadable`
- Components handle their own props and emit events through these hooks
- State management done through copilot hooks rather than local React state

### Authentication Pattern
For backend actions, authentication is handled through:
```typescript
handler: async (params, { getProperties, setState }) => {
  const token = getProperties().authorization;
  // Validate token
  // Update state if authenticated
  setState("key", newValue);
}
```

## Testing Approach

### Unit Tests
- Located in `packages/*/tests/` directories
- Use Jest for JavaScript packages
- Focus on core functionality: action handlers, component behavior, runtime execution

### E2E Tests
- In `examples/e2e/` with Playwright
- Test full integration from frontend to backend
- Focus on user workflows and runtime behavior

### Testing Patterns
- Mock external services (LLMs, MCP servers)
- Test both success and error paths
- Verify state synchronization between frontend/backend

## Gotchas and Non-Obvious Patterns

### Action Handler Parameters
Action handlers take two parameters: `(inputParams, context)`. The context object includes `getProperties()` for accessing frontend props and `setState()` for updating shared state.

### Runtime Action Array Generator
The `actions` property is a generator function `({ properties, url }) => Action[]`, not a static array. This allows conditional action exposure based on frontend context.

### MCP Server Integration
MCP servers require both:
1. Runtime configuration: `mcpServers: [{ endpoint: "...", apiKey: "..." }]`
2. Client factory: `createMCPClient: async (config) => new MCPClient(config)`

### State Management
- Use `setState()` in action handlers, not direct mutations
- State is automatically synced across all connected clients
- State keys should be descriptive and avoid collisions

### Hook Dependencies
React hooks like `useCopilotChat` initialize properties. Make sure to handle async initialization properly, especially for MCP servers.

### Error Handling
- Backend errors thrown in handlers are sent to frontend
- Use try/catch in handlers for user-friendly error messages
- Runtime errors are logged and handled gracefully

## Naming Conventions

### Actions
- `name`: kebab-case, e.g., `send-email`, `query-database`
- Describes what the action does, not how it works

### Components and Hooks
- Standard React conventions: PascalCase for components, camelCase for hooks
- Prefix custom: `CopilotKit`, `CopilotChat`, `useCopilotAction`

### Files and Folders
- kebab-case for folders: `direct-to-llm/`, `backend-actions/`
- camelCase/pascalCase for file names based on content

## Security Considerations

### Authentication
- Always validate JWT tokens in backend actions
- Use `getProperties().authorization` to access tokens
- Never trust frontend-only security

### State Updates
- Only allow authenticated users to update sensitive state
- Validate input parameters before processing
- Use server-side validation for all user inputs

### API Keys
- Store MCP server API keys securely
- Don't log sensitive information in error messages
- Use environment variables for secrets, not hardcoded values