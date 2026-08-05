# Claude response tuning with Bedrock Luna

This repository contains a sanitized, one-call response formatter for Claude
Code Desktop. Claude performs the investigation and tool use; the local Python
wrapper sends the completed draft to Luna through Amazon Bedrock for a
Codex-style rewrite.

## Contents

- `bin/response-rewrite` — fail-safe Python 3 wrapper.
- `output-styles/response-rewrite.md` — strict Claude output protocol.
- `claude-response-tuning.md` — installation, configuration, verification, and
  troubleshooting guide.

The repository contains no credentials. Each user authenticates with their
existing AWS credential chain and configures their Claude settings. Read the
guide before installing.

## Architecture

```mermaid
flowchart LR
    A[Claude tools and reasoning] --> B[Completed draft]
    B --> C[Claude Bash tool]
    C --> D[Python response-rewrite wrapper]
    D --> E[One Bedrock Luna Responses API request]
    E --> F[Codex-style rewritten response]
    F --> G[Deterministic integrity checks]
    G -->|valid| H[stdout]
    G -->|invalid| I[Original draft fallback]
    H --> J[Claude publishes stdout only]
    I --> J
```

The Bash tool is only the launcher and transport layer. The wrapper reads the
draft, builds the JSON request, calls Luna once, validates protected literals,
and returns either the rewrite or the original draft.

## Request boundary

```mermaid
sequenceDiagram
    participant Claude as Claude model
    participant Bash as Bash tool
    participant Wrapper as Python wrapper
    participant Bedrock as Bedrock mantle endpoint

    Claude->>Bash: Write temporary draft and invoke wrapper
    Bash->>Wrapper: Pass draft through stdin
    Wrapper->>Wrapper: Resolve AWS credentials and protect literals
    Wrapper->>Bedrock: POST draft plus editing instructions
    Bedrock-->>Wrapper: One rewritten response
    Wrapper->>Wrapper: Validate code, URLs, paths, numbers, tables
    Wrapper-->>Bash: Rewritten stdout or original-draft fallback
    Bash-->>Claude: Tool result
    Claude->>Claude: Publish stdout byte-for-byte
```

See [claude-response-tuning.md](claude-response-tuning.md) for installation and
verification details.

## Configuration reference panels

These are sanitized, portable reference panels rather than live desktop
captures. They show the exact configuration locations without exposing AWS
credentials or Claude conversation data.

![Output style configuration](screenshots/output-style.svg)

![Claude settings configuration](screenshots/settings-json.svg)

![Per-user file layout](screenshots/file-layout.svg)
