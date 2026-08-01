# Claude response tuning with OpenAI Luna

This repository contains a sanitized, one-call response formatter for Claude
Code Desktop. Claude performs the investigation and tool use; the local Python
wrapper sends the completed draft to OpenAI Luna for a Codex-style rewrite.

## Contents

- `bin/response-rewrite` — fail-safe Python 3 wrapper.
- `output-styles/response-rewrite.md` — strict Claude output protocol.
- `claude-response-tuning.md` — installation, configuration, verification, and
  troubleshooting guide.

The repository contains no API keys. Each user must configure their own key and
Claude settings. Read the guide before installing.
