# starling-bank-mcp

MCP server for Starling Bank API integration, providing tools to interact with Starling Bank's developer API for account management and transactions.

https://github.com/user-attachments/assets/c2b23c22-bd23-487e-a4f5-c62e02280052

This is a 3rd party integration, and is not affiliated with Starling Bank.

> [!WARNING]
> At time of writing, models make frequent mistakes and are vulnerable to prompt injections. As this MCP server gives the model some control of your bank account, mistakes could be costly. Use with caution and at your own risk.

## Installation

Follow the instructions on [install-mcp](https://adamjones.me/install-mcp/?config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsInN0YXJsaW5nLWJhbmstbWNwIl0sIm5hbWUiOiJzdGFybGluZy1iYW5rIiwiZW52Ijp7IlNUQVJMSU5HX0JBTktfQUNDRVNTX1RPS0VOIjoiZXlKaGIuLi4ifX0=), which generates the right config for your MCP client (Claude Code, Claude Desktop, Cursor, Cline, VS Code, and more).

You'll need a Starling Bank personal access token. To create one:
- [Sign up for a Starling Developers account](https://developer.starlingbank.com/signup)
- [Link your Starling Bank account to your Starling Developer account](https://developer.starlingbank.com/settings/account)
- [Create the access token](https://developer.starlingbank.com/personal/token), selecting the scopes based on what you want the AI system to be able to access

Set it as `STARLING_BANK_ACCESS_TOKEN` (replacing the placeholder in the generated config). It'll probably begin something like `eyJhbGciOiJQUzI1NiIsInppcCI6IkdaSVAifQ.`, and be moderately long.

If you want to be able to send payments, also see [PAYMENT_SIGNING_SETUP.md](./PAYMENT_SIGNING_SETUP.md).

## Advanced: HTTP Transport

By default, the server uses stdio transport (for Claude Desktop, Cursor, etc.). You can also run it as an HTTP server:

```bash
STARLING_BANK_ACCESS_TOKEN=eyJhb... MCP_TRANSPORT=http PORT=3000 npx starling-bank-mcp
```

The MCP endpoint will be available at `http://localhost:3000/mcp`.

HTTP mode requires an `MCP_AUTH_TOKEN` (generate one with `openssl rand -hex 32`). Clients must send it as an `Authorization: Bearer <token>` header. If you've secured the server some other way, you can set `MCP_ALLOW_UNAUTHENTICATED=true` instead.

> [!WARNING]
> Without `MCP_AUTH_TOKEN`, other processes on your machine (including websites in your browser) could access the endpoint and control your bank account.

## Advanced: Docker / Coolify

The repo includes a `Dockerfile` (HTTP transport on port 3000, with a `/health` endpoint) and a `docker-compose.yaml`.

**Coolify:** create a new resource from this Git repo, then either:
- choose the **Docker Compose** build pack (recommended). Coolify will prompt for the env vars below. Assign a domain to the `starling-bank-mcp` service.
- or choose the **Dockerfile** build pack, set the exposed port to `3000` and add the env vars manually.

| Variable | Required | Notes |
| --- | --- | --- |
| `STARLING_BANK_ACCESS_TOKEN` | Yes | Personal access token |
| `MCP_AUTH_TOKEN` | Yes | Bearer token clients must send |
| `STARLING_BANK_PRIVATE_KEY_UID` | No | For `payment_create`, see [PAYMENT_SIGNING_SETUP.md](./PAYMENT_SIGNING_SETUP.md) |
| `STARLING_BANK_PRIVATE_KEY_PEM` | No | Multiline, or one line with `\n` escapes |

Then connect a client, for example with Claude Code:

```bash
claude mcp add --transport http starling https://your-domain.example/mcp --header "Authorization: Bearer <MCP_AUTH_TOKEN>"
```

Note: claude.ai web custom connectors can't send bearer headers, so they won't work with this setup.

## Contributing

Pull requests are welcomed on GitHub! To get started:

1. Install Git and Node.js
2. Clone the repository
3. Install dependencies with `npm install`
4. Run `npm run test` to run tests
5. Build with `npm run build`

## Releases

Versions follow the [semantic versioning spec](https://semver.org/).

To release:

1. Use `npm version <major | minor | patch>` to bump the version
2. Run `git push --follow-tags` to push with tags
3. Wait for GitHub Actions to publish to the NPM registry.
