# Agent-Test Workspace Guidelines

## Purpose

This workspace is a dedicated test harness for using the sibling `kipilot-mcp` server from VS Code through MCP.

## Workflow Expectations

- Use the configured `kipilot-mcp` MCP server for KiCad tasks.
- Before any board-specific reasoning, verify that the `kipilot-mcp` MCP tools are actually available in the session. If the tool namespace is missing, treat that as an MCP server startup/configuration problem, not as a signal to search the repository for board state.
- Verify connectivity with `ping_kicad` or `get_kicad_version` before board-specific actions when session state is unknown.
- Prefer board inspection first, then `dry_run=true`, then live mutations only if the user explicitly asks for a real write.
- Prefer exact IDs returned by read tools for follow-up updates or deletions.
- Prefer narrow MCP queries and smaller limits over broad fetches that force follow-up reads of generated chat resource files.
- For free-form board text edits, prefer `kicad_get_board_text` before title block, project-variable, or repository-file reasoning.
- If a specialized KiPilot write tool contradicts earlier read-tool results about the same item, do one narrow disambiguation step instead of repeating the same failing write.
- If an equivalent narrow MCP fallback exists, prefer that fallback over declaring the capability missing.
- If the user mentions `F.SilkS`, `B.SilkS`, silkscreen, logo artwork, or printed board text, distinguish that from footprint placement side before choosing a mutation path.
- Treat `kicad_find_footprints(text_query=...)` as a lookup over footprint `reference`, `value`, and `id`, not as proof that the matching visible object is standalone silkscreen content.
- If the user explicitly says `footprint with value ...`, prioritize footprint lookup over standalone board text or graphics queries.
- Do not send guessed raw numeric layer IDs in MCP calls unless those IDs were confirmed from the live board.

## Scope Limits

- Treat this workspace as KiCad 10 PCB-first.
- Do not assume schematic automation, plotting, export automation, or headless KiCad control are available.
- If the requested action is outside the current MCP surface, say that clearly and offer a PCB-scoped alternative.
- When the sibling `kipilot-mcp` source is available in the workspace and an MCP result looks like a local server bug, you may inspect and patch that local server to unblock the requested PCB task.
- Footprint read results now include a compact `child_graphics` layer summary for footprint-internal non-pad artwork/text, so use that summary or the flip result to verify mirrored silkscreen movement after a footprint side flip.

## Safety

- Treat `kicad_revert_board` and `kicad_delete_items` as destructive operations.
- Do not claim that a board change was applied unless the MCP tool returned `ok: true`.
- If mutations are disabled, use dry-run previews and explain that live writes are currently blocked by configuration.
- Do not change `.vscode/mcp.json` or other workspace configuration just to enable live writes unless the user explicitly asks for that workspace change.
- When inferring function blocks from footprints, nets, or zones, clearly mark them as inference rather than direct fact.
- Do not claim that a local KiPilot server fix changed the board; after patching the local server, require a server restart and a fresh MCP write result before reporting success.

## Debugging

- If the MCP server is unavailable, help diagnose the workspace setup instead of guessing.
- If `ping_kicad` or `get_kicad_version` cannot even be executed because the MCP server or tool namespace is unavailable, stop board analysis immediately and report that the MCP server is not running or not connected in the host session.
- Check `.vscode/mcp.json`, KiCad runtime state, and the workspace log path before proposing broader changes.
- Do not inspect `.kicad_pcb` files or other workspace artifacts to answer live-board questions when the MCP server is unavailable.
- Do not inspect VS Code chat-session artifacts such as generated `content.json` files when a narrower MCP query can provide the same answer.
- If a KiPilot tool failure contradicts confirmed live board data and the server source is present locally, debug the local server code before concluding that the operation is fundamentally unsupported.