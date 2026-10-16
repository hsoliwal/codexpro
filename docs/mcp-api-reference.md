# MCP operation reference

This compact reference covers the 33 registration sites in [server.ts](../src/server.ts) at the [pinned baseline](architecture-and-projects.md). Complete handler bodies were read for this reference. It is not a generated exhaustive JSON Schema listing, a claim that every branch was tested, or a replacement for live tool discovery and the source schemas.

## Common calling and result rules

- Workspace operations generally accept optional `workspace_id`; omission uses the MCP session's selected workspace. `open_current_workspace` selects the launch root; `open_workspace` can change the selection only within configured allowed roots.
- Schema validation happens before handlers. Missing required fields, invalid enums/ranges, and helper failures become `isError: true` tool results. Many Git/diagnostic operations instead return successful envelopes with explicit error/status fields; inspect those too.
- Normal results have text `content`, `structuredContent`, and tool-name/title tags. `view_image` additionally returns native MCP image content. Known secret and path masking is heuristic; inspect [data boundaries](workspace-and-analysis-contracts.md).
- Optional presentation metadata depends on tool cards. Structured view truncation is not necessarily the same as analysis or input truncation. Returned coverage/warnings/limits belong to the API contract.
- Source strings are not automatically safe commands. Handoff plans, loaded skills, model completions, transcript messages, and reviewer verdicts remain data with distinct trust and execution boundaries.

In the table, **M** means minimal/standard/full; **S** means standard/full; **F** means full only. Further write/Bash/analysis/history/connection-test gates apply. `codexpro` does not bypass these gates.

## Operations

| Operation / base surface | Inputs and defaults | Outputs and important behavior |
| --- | --- | --- |
| `codexpro` / M | Optional `action`, nested `args`; default list_actions; aliases normalize spaces/hyphens/case | Lists actual registered actions or calls a registered validated handler. Self-recursion and disabled/unregistered actions fail; wrapper adds action identity |
| `fabric` / S | Required `action`: invariants, dag_contract, dag_execute, chrome_contract, chrome_status, chrome_complete, chrome_summarizer_contract, chrome_summarize, chrome_summarize_document, chrome_summarize_corpus; action-specific fields | [Compute contracts](compute-contracts.md) define bounds. Can launch Java or call loopback HTTP. Document/corpus actions first read guarded valid UTF-8 workspace files; declared read-only hints do not mean no process/network activity |
| `server_config` / M | No arguments | Masked config/limits and registeredTools plus search/Bash/Git runtime probes. Does not return the auth token; probes are more than reading static config |
| `codexpro_self_test` / M | Optional workspace; write_probe, bash_probe, pro_context_probe default true; include_global_skills true; max_skills 40, cap120 | Diagnostic checks/counts/status and files_touched. May temporarily write/edit a context probe, run shell probes, and scan global skills. Cleanup attempts restore original probe bytes/mode/timestamps or remove a new probe; failures are reported. This is not a no-side-effect metadata request |
| `codexpro_inventory` / F | Optional workspace; include_global_skills and include_mcp_servers true; max_skills120, cap500 | Skill and configured MCP-server names/counts plus modes. Global file reads possible; discovered servers are not connected |
| `load_skill` / S | Required exact `name`; optional source/path override from inventory, include_global_skills, max_skills500, max_bytes40,000 (1,000–100,000) | Bounded selected SKILL.md and source metadata/bytes/truncation. Default global scanning depends on source/path; does not execute instructions or accept arbitrary file reads |
| `list_workspaces` / F | No arguments | Manager-local opened/adopted workspaces/count and current selected ID. A shared-registry workspace is added locally when explicitly resolved; listing does not enumerate every HTTP session's workspace |
| `open_current_workspace` / M | include_tree false, max_depth2 (1–8), include_skills/global false | Selects configured default and returns ID/root/instructions/skills/tree/Git/modes. Explicitly disables context bootstrap |
| `open_workspace` / M | Optional root or path alias; both must match if supplied. include_tree true, max_depth3, max_files500 (cap3000), skills/global false | Opens/selects canonical allowed directory and returns orientation fields. Deprecated bootstrap_context is ignored; no context bootstrap |
| `workspace_snapshot` / F | Optional workspace; max_depth3, max_files500, skills/global false | Summary plus existing context-file list. Does not change selected root through a root argument |
| `inspect_workspace` / S | Optional workspace/path; include_symbols/relationships true; returned file/symbol/relationship limits | Bounded analysis with schema_version, files, graph, coverage, warnings/cache. Path emphasizes/filters an already-built workspace analysis, rather than restricting all source scanning. Widget mode imposes smaller returned views |
| `tree` / S | path `.`, max_depth4 (1–12), include_hidden false, max_entries800 (cap3000) | Guarded directory tree and traversal result; hidden inclusion does not bypass blocked globs |
| `search` / S | Required query; regex false, path `.`, optional glob/intent/symbol, hidden false, result cap/config, include_tests optional | matches/truncated/backend and optional analysis. Regex requires ripgrep; structured symbol intent can differ from lexical matching |
| `read` / M | Required path; optional start_line/end_line ≥1 and max_bytes within schema/config bounds | Numbered text, actual range/total lines, bytes/hash; does not promise byte-identical display of original line endings |
| `view_image` / S | Required path; optional max_bytes4096–2,000,000 | Text plus base64 MCP image; structured MIME/dimensions/bytes/hash. Signature checks are not complete image decoding |
| `write` / M + workspace writes | Required path/content; create_dirs and overwrite true; optional expected_sha256 | path/existed/bytes/new hash/diff stats/diff. Invalidates analysis cache if changed; optional hash rejects stale content; existing writes are not rollback transactions |
| `edit` / M + workspace writes | Required path/old_text/new_text; replace_all false; optional positive expected_replacements and expected_sha256 | replacement count, bytes/hash/diff. Exact-match semantics; invalidates analysis if changed |
| `apply_patch` / M + workspace writes | Required unified patch | paths, Git stdout/stderr, diff/stats/changed; guard/mode/secret/symlink checks followed by git check/apply; invalidates analysis if changed |
| `import_file` / M + workspace writes | Required file object `{download_url,file_id}` and destination; optional MIME/name/expected_sha256; overwrite false | path, bytes/hash, declared/detected MIME/status, verified flag, source file metadata/overwritten. Network/temp/destination writes; invalidates analysis |
| `bash` / M + Bash enabled | Required command; optional cwd, session_id, timeout_ms; default30s/config cap | Command/cwd/runtime/session/exit/signal/duration/stdout/stderr. Compact transcript hides raw streams from prose but retains structured output. Nonzero exit is not automatically an MCP error |
| `git_status` / F | Optional workspace/path | status/status_error, changed_files/changed. Changed entries are status lines, not uniformly bare filenames |
| `git_diff` / F | Optional path, staged false, include_diff true | Diff/stats/changed and diff_error. Stats-only uses a different Git helper; raw errors are not equivalent to clean state |
| `show_changes` / M | Optional path, staged false, include_diff true, since last_shown, mark_reviewed true | Git status/diff/stats plus optional heuristic impact. Maintains server-local review checkpoints; repeated identical reviews may return changed=false and empty diff/changed_files even with workspace changes still present |
| `read_handoff` / S | Optional workspace | Existing context files/count/text/preview; does not start an executor |
| `wait_for_handoff` / S | Optional expected plan_hash, since_iteration; max_wait_seconds20 (1–60), poll_ms1000 (250–5000); diff/log/test excerpts true | Polls bounded state/excerpts; awaited_terminal differs from awaited_completed/succeeded. Completed, failed, timed_out, interrupted, and orphaned are terminal. Does not launch/retry an agent |
| `codex_context` / F | target_path `.`, include_ai_bridge/git true, include_diff false, max_agent_bytes60,000 | Instruction-chain/context file lists and optional Git context; returned instructions are source content |
| `export_pro_context` / S | Optional title/selected_paths/extra_globs; automatic important/changed/diff/context inclusion true; max_depth3/max_files24/max_file_bytes60,000 | Writes context bundle and returns path/bytes/included/skipped/truncated. No automatic model upload; generic source-write off does not disable this workflow |
| `codex_sessions` / history metadata or read | max_sessions30 (cap200), optional query | Saved-file metadata/list roots/matched count; no workspace_id and no live-conversation attachment |
| `read_codex_session` / history read | session_id or source_path; direction tail; optional cursor; max_messages80/max_total_bytes80,000; tool outputs included, cap20,000 each | Chronological page, cursors/more/truncation/source size; history-root guard is separate from workspace roots |
| `search_codex_session` / F + history read | session/source, required query; optional roles/tool_names/time bounds; max_results30/max_snippet_bytes600 | Byte-offset match IDs/snippets and truncation. Despite the option name, snippet helper measures characters. Name-list metadata includes it in read mode more broadly than actual registration |
| `read_codex_session_around` / F + history read | session/source plus message_id/byte_offset/timestamp; before/after10, content80,000 and tool-output20,000 defaults | Target/context messages and more flags; same actual registration caveat as history search |
| `handoff_to_agent` / S, also minimal in handoff write mode | Required plan; optional agent custom, agent_name/model/title, append false | Writes plan/context/logs and returns artifact paths/diff. Does not execute agent; multiple writes can partially succeed |
| `handoff_to_codex` / F | Required plan, optional title/append | Compatibility plan-writing wrapper with agent=codex; no Codex execution |

## Mode and lifecycle details

Connection-test mode removes the supertool, fabric, self-test, generic writes/import, Bash, export, and handoff writers. It does not replace every other configuration value with a new security identity. Session-history access remains separately opt-in; actual discovery should be checked after any mode change/restart.

`show_changes` keys checkpoints by workspace, normalized scoped path, and staged flag. Its fingerprint uses status/diff plus bounded untracked-file evidence. `include_diff=false` still obtains the underlying diff in this handler, but does not write a review checkpoint. `mark_reviewed` describes a display checkpoint, not human approval, code acceptance, or a Git commit. Use `since=workspace` for a fresh whole-current-change view.

`wait_for_handoff` reads the state file repeatedly, treats parse/read failure as missing state, and can infer orphaned in-flight work from recorded PIDs. PID existence does not establish process identity or successful completion. Expected plan hash and iteration constrain which terminal run satisfies the wait. Excerpt paths are restricted back to the context directory and passed through the path guard. Failed/interrupted/orphaned work may require reconciliation before retry because recorded state cannot undo external effects.

## Contract verification limits

Every registration site is listed, but this compact table intentionally omits some optional presentation fields and does not replace full schema/inline documentation. Remaining acceptance work includes all mode combinations, SDK compatibility paths, branch-specific failures, schema/implementation discrepancies, and live transport/platform tests. [Coverage](documentation-coverage.md) therefore remains partial.
