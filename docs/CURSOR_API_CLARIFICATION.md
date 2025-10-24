# Cursor API Key Clarification

## Important: "Cursor API" is Actually Anthropic API

If you're using Cursor and have a "Cursor API key", it's important to understand that:

### The Truth About "Cursor API Keys"

**Cursor does not provide its own AI API.** Instead, Cursor uses **Anthropic's Claude models** under the hood.

What people commonly refer to as a "Cursor API key" is actually:
- **An Anthropic API key** from https://console.anthropic.com/
- Used by Cursor to access Claude models
- Works with the standard Anthropic API endpoints

### Why the Confusion?

Many users (including the FinOps project example) name their environment variable `CURSOR_API_KEY` for convenience:

```bash
# This is actually an Anthropic API key!
CURSOR_API_KEY=sk-ant-api03-your-anthropic-key-here
```

But this is just a naming convention. The key itself is an **Anthropic API key**.

## How to Configure Task Master Correctly

### For MCP Usage (Cursor, VS Code, etc.)

In your MCP configuration file (e.g., `~/.cursor/mcp.json`), Task Master needs the key under **`ANTHROPIC_API_KEY`**:

```json
{
  "mcpServers": {
    "task-master-ai": {
      "command": "/path/to/node",
      "args": ["/path/to/ai-task-master/dist/mcp-server.js"],
      "cwd": "/path/to/ai-task-master",
      "env": {
        "ANTHROPIC_API_KEY": "sk-ant-api03-your-key-here"
      }
    }
  }
}
```

**Note**: You can include both keys with the same value if needed:

```json
"env": {
  "ANTHROPIC_API_KEY": "sk-ant-api03-your-key-here",
  "CURSOR_API_KEY": "sk-ant-api03-your-key-here"
}
```

### For CLI Usage

In your project's `.env` file:

```bash
# Required for Anthropic provider
ANTHROPIC_API_KEY=sk-ant-api03-your-key-here
```

### In Project Configuration

In `.taskmaster/config.json`, use the **`anthropic`** provider:

```json
{
  "models": {
    "main": {
      "provider": "anthropic",
      "modelId": "claude-3-5-sonnet-20241022",
      "maxTokens": 8192,
      "temperature": 0.2
    },
    "research": {
      "provider": "anthropic",
      "modelId": "claude-3-opus-20240229",
      "maxTokens": 4096,
      "temperature": 0.1
    },
    "fallback": {
      "provider": "anthropic",
      "modelId": "claude-3-5-sonnet-20241022",
      "maxTokens": 16384,
      "temperature": 0.2
    }
  }
}
```

## The "cursor" Provider in Task Master

Task Master does have a "cursor" provider option in its configuration, but this is **experimental and not recommended** because:

1. **There is no public Cursor API endpoint** (`api.cursor.sh` is not publicly accessible)
2. **It causes connection errors** (`ENOTFOUND` errors)
3. **It's unnecessary** - Use the `anthropic` provider directly instead

### Why Does the "cursor" Provider Exist?

The "cursor" provider was created as an attempt to support Cursor users, but it was based on a misunderstanding:
- It tries to connect to `https://api.cursor.sh/v1` (which doesn't exist publicly)
- It's configured in `src/ai-providers/cursor.js`
- It's not functional for most users

## Correct Provider Mapping

| What You Have | What You Should Use |
|---------------|---------------------|
| Anthropic API key | `anthropic` provider |
| "Cursor API key" (from Anthropic) | `anthropic` provider |
| OpenAI API key | `openai` provider |
| Perplexity API key | `perplexity` provider |

## Example: FinOps Project Pattern

The FinOps project uses this pattern successfully:

```javascript
// backend/controllers/costAnalysisController.js
const Anthropic = require('@anthropic-ai/sdk');

function getAnthropicClient(apiKey) {
  // Accepts key from multiple sources for convenience
  const key = apiKey || 
    process.env.CURSOR_API_KEY ||  // For users who named it this
    process.env.ANTHROPIC_API_KEY; // Standard name
  
  if (!key) return null;
  
  return new Anthropic({
    apiKey: key,
  });
}
```

This works because both environment variable names contain the **same Anthropic API key**.

## Getting an Anthropic API Key

1. **Visit Anthropic Console**: https://console.anthropic.com/
2. **Create Account**: Sign up or sign in
3. **Generate API Key**:
   - Go to "API Keys" section
   - Click "Create Key"
   - Copy the key (starts with `sk-ant-`)
4. **Add to Configuration**:
   - For MCP: Add to `mcp.json` as `ANTHROPIC_API_KEY`
   - For CLI: Add to `.env` as `ANTHROPIC_API_KEY`
5. **Configure Provider**:
   - Set `provider: "anthropic"` in `.taskmaster/config.json`

## Pricing

Anthropic API pricing (as of 2024):
- **Claude 3.5 Sonnet**: $3/M input tokens, $15/M output tokens
- **Claude 3 Opus**: $15/M input tokens, $75/M output tokens
- **Claude 3 Haiku**: $0.25/M input tokens, $1.25/M output tokens

Typical Task Master usage: ~$0.01-0.05 per task generation

## Troubleshooting

### Error: "Cannot connect to API: getaddrinfo ENOTFOUND api.cursor.sh"

**Cause**: You're using `provider: "cursor"` in config.json

**Solution**: Change to `provider: "anthropic"`

### Error: "Anthropic API key is required"

**Cause**: The `ANTHROPIC_API_KEY` is not set or is a placeholder

**Solution**: 
1. Check your MCP config has `ANTHROPIC_API_KEY` with a real key
2. Restart Cursor to reload MCP configuration
3. Verify the key starts with `sk-ant-`

### Error: "All roles in the sequence [research, fallback, main] failed"

**Cause**: All three provider roles failed to connect (likely all using invalid "cursor" provider)

**Solution**:
1. Update all roles in `.taskmaster/config.json` to use `"provider": "anthropic"`
2. Ensure `ANTHROPIC_API_KEY` is set in MCP config
3. Restart Cursor

## Summary

✅ **DO**:
- Use `provider: "anthropic"` in Task Master configuration
- Set `ANTHROPIC_API_KEY` in your MCP config or `.env` file
- Get your API key from https://console.anthropic.com/

❌ **DON'T**:
- Use `provider: "cursor"` (it won't work)
- Think "Cursor API" is different from "Anthropic API"
- Try to use `api.cursor.sh` endpoint

**Remember**: What Cursor users call a "Cursor API key" is actually an Anthropic API key. Use it with the `anthropic` provider in Task Master.

---

**Related Documentation**:
- [MCP API Key Setup Guide](./MCP_API_KEY_SETUP.md)
- [Configuration Guide](./configuration.md)
- [Provider Setup](./providers/)

