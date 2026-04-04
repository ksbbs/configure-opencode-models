---
name: configure-opencode-models
description: Configure custom providers and models in opencode.json, including context window, output limits, and input modalities. Use when the user wants to add or modify AI model configurations for OpenCode CLI.
---

# Configure OpenCode Models

Help users configure custom providers and models in the OpenCode configuration file.

## Configuration File Location

The OpenCode config file is located at:
- Windows: `C:\Users\<username>\.config\opencode\opencode.json`
- macOS/Linux: `~/.config/opencode/opencode.json` or `~/.opencode.json`

## JSON Schema

Always add the `$schema` field for IDE autocomplete:
```json
{
  "$schema": "https://opencode.ai/config.json"
}
```

## Complete Configuration Structure

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5",
  "mcp": {
    "<server-name>": {
      "type": "local",
      "command": ["npx", "-y", "<package>"],
      "enabled": true,
      "environment": {}
    }
  },
  "plugin": ["plugin-name@latest"],
  "provider": {
    "<provider-name>": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "apiKey": "<your-api-key>",
        "baseURL": "<api-endpoint>"
      },
      "models": {
        "<model-id>": {
          "name": "<display-name>",
          "limit": {
            "context": <context-window-size>,
            "output": <max-output-tokens>
          },
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          },
          "attachment": <true|false>
        }
      }
    }
  }
}
```

---

## MCP Server Configuration

MCP servers are defined in the `mcp` field. Each server needs a unique name.

### Local MCP Server

```json
{
  "mcp": {
    "github": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-github"],
      "enabled": true,
      "environment": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
      }
    }
  }
}
```

### Remote MCP Server

```json
{
  "mcp": {
    "sentry": {
      "type": "remote",
      "url": "https://mcp.sentry.dev/mcp",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer {env:SENTRY_AUTH_TOKEN}"
      }
    }
  }
}
```

### MCP Server Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `type` | string | Yes | `"local"` or `"remote"` |
| `command` | array | Yes (local) | Command and arguments to run the server |
| `url` | string | Yes (remote) | Remote MCP server URL |
| `enabled` | boolean | No | Enable or disable the server |
| `environment` | object | No | Environment variables for local servers |
| `headers` | object | No | HTTP headers for remote servers |
| `timeout` | number | No | Timeout in ms (default: 5000) |

---

## Provider Configuration

### Provider Options

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}",
        "baseURL": "https://api.anthropic.com/v1",
        "timeout": 600000,
        "setCacheKey": true
      }
    }
  }
}
```

### Provider Fields

| Field | Description |
|-------|-------------|
| `npm` | AI SDK package name (e.g., `@ai-sdk/openai-compatible`) |
| `name` | Display name in UI |
| `options.apiKey` | API key (use `{env:VAR_NAME}` for env vars) |
| `options.baseURL` | Custom API endpoint |
| `options.timeout` | Request timeout in ms (default: 300000) |
| `options.headers` | Custom HTTP headers |

---

## Model Configuration

### Model Fields

```json
{
  "models": {
    "my-model": {
      "name": "My Model Display Name",
      "id": "actual-model-id",
      "limit": {
        "context": 200000,
        "output": 65536
      },
      "modalities": {
        "input": ["text", "image"],
        "output": ["text"]
      },
      "attachment": true
    }
  }
}
```

### limit.context
- Context window size in tokens
- Common values:
  - 128k (131072)
  - 200k (200000)
  - 256k (256000)
  - 1M (1000000)

### limit.output
- Maximum output tokens
- Recommended values:
  - 32k (32768) - standard
  - 64k (65536) - generous
  - 128k (131712) - unlimited feel

### modalities.input
- Array of supported input types
- Options: `["text"]`, `["text", "image"]`, `["text", "image", "audio", "video", "pdf"]`

### attachment
- `true` for multimodal models (supports image/file uploads)
- `false` for text-only models

---

## Common Provider Examples

### OpenAI-Compatible API (Infini, etc.)

```json
{
  "provider": {
    "infini": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "apiKey": "sk-xxx",
        "baseURL": "https://cloud.infini-ai.com/maas/coding/v1"
      },
      "models": {
        "glm-5": {
          "name": "glm-5",
          "limit": { "context": 200000, "output": 65536 },
          "modalities": { "input": ["text"], "output": ["text"] },
          "attachment": false
        },
        "kimi-k2.5": {
          "name": "kimi-k2.5",
          "limit": { "context": 256000, "output": 65536 },
          "modalities": { "input": ["text", "image"], "output": ["text"] },
          "attachment": true
        }
      }
    }
  }
}
```

### Local Model (LM Studio, Ollama)

```json
{
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": {
        "baseURL": "http://127.0.0.1:1234/v1"
      },
      "models": {
        "local-model": {
          "name": "Local Model"
        }
      }
    }
  }
}
```

### Ollama

```json
{
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "llama2": {
          "name": "Llama 2"
        }
      }
    }
  }
}
```

---

## Variables

Use variables to reference environment variables and file contents:

### Environment Variables

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

### File Contents

```json
{
  "provider": {
    "openai": {
      "options": {
        "apiKey": "{file:~/.secrets/openai-key}"
      }
    }
  }
}
```

---

## Workflow

1. Read the existing config file
2. Ask user for:
   - Provider name and API endpoint
   - API key (if new provider)
   - Model IDs and their specifications
3. Merge/update the configuration
4. Write back the config file with proper formatting (2-space indentation)
5. Summarize changes made

## Notes

- Preserve existing MCP server configurations
- Don't expose API keys in summaries
- Validate JSON syntax before writing
- Use standard 2-space indentation
- MCP server names should be meaningful (e.g., `github`, `zeabur`, not `mcpServers`)
- Common multimodal models: GPT-4o, Claude 3.5, Gemini, Kimi
- Common text-only models: GPT-4, DeepSeek, GLM

## Official Documentation

- Config: https://opencode.ai/docs/zh-cn/config/
- Providers: https://opencode.ai/docs/zh-cn/providers/
- MCP Servers: https://opencode.ai/docs/zh-cn/mcp-servers/
