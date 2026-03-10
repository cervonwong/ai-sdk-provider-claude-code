# Security Policy

## Security Model

`ai-sdk-provider-claude-code` is a TypeScript library that bridges the [Vercel AI SDK](https://sdk.vercel.ai/) with the [Claude Agent SDK](https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk). It spawns and communicates with the Claude Code CLI process. Understanding its security model is essential before integrating it into your application.

### Trust Boundaries

1. **Application developer (trusted)**: Settings such as `pathToClaudeCodeExecutable`, `spawnClaudeCodeProcess`, `env`, `sdkOptions`, `plugins`, `extraArgs`, and `allowDangerouslySkipPermissions` are **developer-controlled configuration**. These options intentionally provide powerful control over the underlying CLI process. **Never expose these settings to untrusted end-user input.**

2. **Claude Code CLI process**: The provider spawns a child process running the Claude Code CLI. The CLI has access to the local filesystem, network, and system commands subject to its own permission model. The `permissionMode` and `allowedTools` / `disallowedTools` settings restrict what the CLI can do.

3. **MCP servers**: When configured via `mcpServers`, external Model Context Protocol servers may execute tools with access to external resources. Only configure MCP servers you trust.

### Dangerous Settings

The following settings grant elevated privileges and should be used with caution:

| Setting | Risk | Guidance |
|---------|------|----------|
| `allowDangerouslySkipPermissions: true` | Disables **all** permission checks for the CLI process | Only use in fully trusted, sandboxed environments |
| `permissionMode: 'bypassPermissions'` | Bypasses the permission prompt | Combine with `allowedTools` to limit scope |
| `spawnClaudeCodeProcess` | Allows arbitrary process spawning | Never accept from untrusted input |
| `pathToClaudeCodeExecutable` | Controls which binary is executed | Only use absolute paths to verified binaries |
| `plugins` | Loads and executes code from local paths | Only load plugins you have reviewed |
| `extraArgs` | Passes arbitrary CLI arguments | Validate arguments if derived from user input |
| `sdkOptions` | Escape hatch that can override internal settings | Review carefully; some fields are blocked |
| `env` | Sets environment variables for the child process | Do not pass secrets unnecessarily |
| `debugFile` | Writes debug output (may contain sensitive data) to a file | Use only in development; restrict path |

### Environment Variable Handling

The provider uses an **allowlist** approach for environment variables passed to the child process (see `getBaseProcessEnv()` in source). Only essential system variables (e.g., `PATH`, `HOME`, `SHELL`) and Claude-specific variables are forwarded. User-supplied `env` and `sdkOptions.env` values are merged on top of this base and may override allowlisted values.

## Supported Versions

| Version | Supported |
|---------|-----------|
| 3.x.x | ✅ |
| 2.x.x | ⚠️ Security fixes only |
| < 2.0 | ❌ |

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly:

1. **Do NOT open a public issue.**
2. Open a [GitHub Security Advisory](https://github.com/ben-vargas/ai-sdk-provider-claude-code/security/advisories/new) (preferred), or contact the maintainer via the email listed in the [npm package page](https://www.npmjs.com/package/ai-sdk-provider-claude-code).
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)
4. You should receive an acknowledgment within 72 hours.
5. A fix will be developed and released as soon as practical, typically within 14 days for critical issues.

## Security Best Practices for Users

1. **Pin dependency versions** in production (use `package-lock.json` or equivalent).
2. **Run `npm audit`** regularly to check for known vulnerabilities in transitive dependencies.
3. **Never pass untrusted user input** directly to provider settings like `env`, `extraArgs`, `plugins`, `pathToClaudeCodeExecutable`, or `spawnClaudeCodeProcess`.
4. **Use the strictest `permissionMode`** that meets your needs. Avoid `bypassPermissions` in production.
5. **Review MCP server configurations** before deploying. Only connect to trusted servers.
6. **Avoid `debugFile` in production** — debug logs may contain prompts, tool results, and other sensitive data.
7. **Keep the Claude Code CLI updated** to benefit from upstream security patches.
