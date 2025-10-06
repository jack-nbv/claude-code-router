# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

-   **Build the project**:
    ```bash
    npm run build
    ```
-   **Start the router server**:
    ```bash
    ccr start
    ```
-   **Stop the router server**:
    ```bash
    ccr stop
    ```
-   **Check the server status**:
    ```bash
    ccr status
    ```
-   **Run Claude Code through the router**:
    ```bash
    ccr code "<your prompt>"
    ```
-   **Release a new version**:
    ```bash
    npm run release
    ```

## Architecture

This project is a TypeScript-based router for Claude Code requests. It allows routing requests to different large language models (LLMs) from various providers based on custom rules.

-   **Entry Point**: The main command-line interface logic is in `src/cli.ts`. It handles parsing commands like `start`, `stop`, and `code`.
-   **Server**: The `ccr start` command launches a server that listens for requests from Claude Code. The server logic is initiated from `src/index.ts`.
-   **Configuration**: The router is configured via a JSON file located at `~/.claude-code-router/config.json`. This file defines API providers, routing rules, and custom transformers. An example can be found in `config.example.json`.
-   **Routing**: The core routing logic determines which LLM provider and model to use for a given request. It supports default routes for different scenarios (`default`, `background`, `think`, `longContext`, `webSearch`) and can be extended with a custom JavaScript router file. The router logic is likely in `src/utils/router.ts`.
-   **Providers and Transformers**: The application supports multiple LLM providers. Transformers adapt the request and response formats for different provider APIs.
-   **Claude Code Integration**: When a user runs `ccr code`, the command is forwarded to the running router service. The service then processes the request, applies routing rules, and sends it to the configured LLM. If the service isn't running, `ccr code` will attempt to start it automatically.
-   **Dependencies**: The project is built with `esbuild`. It has a key local dependency `@musistudio/llms`, which probably contains the core logic for interacting with different LLM APIs.
-   `@musistudio/llms` is implemented based on `fastify` and exposes `fastify`'s hook and middleware interfaces, allowing direct use of `server.addHook`.

## Development vs Production Testing

**Important**: There are two ways to run the router during development. Understanding the difference prevents common mistakes:

### 1. Testing with Installed Command (`ccr`)
```bash
# Build the project
npm run build

# Copy to global npm installation (required to test installed version)
cp ./dist/cli.js "$(npm root -g)/@musistudio/claude-code-router/dist/cli.js"
cp ./dist/index.html "$(npm root -g)/@musistudio/claude-code-router/dist/index.html"

# Then test with the global command
ccr start
ccr code "test prompt"
ccr stop
```

**Use when**: Testing the production/installed behavior, or when changes need to be deployed globally.

### 2. Testing with Local Build (`./dist/cli.js`)
```bash
# Build the project
npm run build

# Run directly from local dist folder
node ./dist/cli.js start
node ./dist/cli.js code "test prompt"
node ./dist/cli.js stop
```

**Use when**: Quickly testing local changes without affecting the global installation.

### Common Pitfall ⚠️
After making code changes and running `npm run build`, the **global `ccr` command will NOT automatically use your new changes**. You must either:
- Copy the built files to the global installation (Option 1 above), OR
- Use `node ./dist/cli.js` directly (Option 2 above)

### Quick Update Script
For rapid development, create an alias in your shell:
```bash
# Add to ~/.bashrc or ~/.zshrc
alias ccr-update='npm run build && cp ./dist/cli.js "$(npm root -g)/@musistudio/claude-code-router/dist/cli.js" && cp ./dist/index.html "$(npm root -g)/@musistudio/claude-code-router/dist/index.html" && ccr stop && ccr start'
```

Then simply run:
```bash
ccr-update  # Rebuilds, updates global install, and restarts
```