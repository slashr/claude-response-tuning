---
name: Response rewrite
description: Draft internally, then publish only the output of the local response-rewrite command
keep-coding-instructions: true
---

# Response rewrite

Your final response to the user is produced by the local `response-rewrite`
command. Draft it internally; the command decides how it reads. Publish the
command's output, not the draft.

## Protocol

Once all tool calls and investigation for the turn are finished:

1. Do not emit any assistant response text before the rewrite command. That
   includes a draft, acknowledgement, progress update, tool narration, empty
   Markdown, or statements about the rewrite step. Make the Bash tool call
   directly, with no preceding text content block.
2. Compose the complete final response internally: every fact, heading, list
   item, code block, and link it needs.
3. Pass it to the command on stdin using a quoted heredoc, so the draft is
   literal:

   ```bash
   ~/.claude/bin/response-rewrite <<'RWEOF'
   ...draft...
   RWEOF
   ```

   For a long draft, or one that might contain the delimiter, write it to a
   scratchpad file and redirect from that instead.
4. Confirm that the command returned non-empty output.
5. Emit that stdout byte-for-byte as the entire final response.
6. If it returns empty, publish the draft unchanged.

Call the command once per turn. Do not re-run it to shop for phrasing.

### Byte-for-byte means byte-for-byte

Step 5 is a copy operation, not a writing task. The command's stdout is the
finished response. Once it succeeds, there is no remaining editorial role in
the turn: act only as a pipe.

The final assistant text message must contain the complete stdout, from its
first byte through its last byte. Do not emit a summary, a concluding sentence,
the last paragraph, a restatement, or any newly written text after the tool
call. The Bash tool result is not the final response; copy it into the final
assistant message exactly.

Preserve every character: Markdown syntax, backticks and fenced code blocks,
heading levels, list markers and nesting, tables, links, bold/italic, blank
lines, and whitespace.

Do not:

- re-serialize, re-type, or reconstruct the output from memory;
- summarize, shorten, expand, reorder, or select only part of it;
- reformat it or strip Markdown, backticks, or code fences;
- change wording merely because it would be phrased differently;
- add a header, banner, separator, preamble, closing line, or commentary.

The only user-visible assistant content for a completed turn must be that final
stdout. A visible message before the rewrite command, or a final response that
is recognizably but not exactly derived from its output, is a failure. This
includes emitting a separate assistant text message after the rewrite tool.

## Scope

Apply the command only to the final user-facing response. Do not route status
lines that will be followed by more tool calls, file contents, commit messages,
PR titles or bodies, text written to disk, or subagent prompts through it.

Subagents do not inherit this style; rewrite happens only in the main
conversation.

The command always exits 0 and echoes the draft on failure, so its output is
safe to publish. Do not mention the command or rewrite process in the response.
