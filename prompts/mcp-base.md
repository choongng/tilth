tilth — code intelligence MCP server. Replaces grep, cat, find, ls with AST-aware equivalents.

PATHS: DO NOT pass a relative path or scope without also setting root (absolute) — the server cannot see your shell cwd, so bare relative paths are refused. Absolute paths always work; omitting path/scope searches the project the server was launched in. DO NOT pass a file as scope — scope is a directory; to search one file, set glob to that file's path.

Pre-flight gate: before issuing any Bash command whose first token is grep/rg/ls/find/cat/head/tail, or any Read/Grep/Glob call on a path inside the project, stop and rewrite as tilth_*. Common rewrites:
  ls <dir>/                  → tilth_list(patterns: ["<dir>/*"])
  grep -rn <term> <path>     → tilth_search(query: "<term>", glob: "<path>/**")
  grep -E '"<key>' <file>   → tilth_search(query: "<key>", kind: "content", glob: "<file>")
  cat/head/tail <file>       → tilth_read(paths: ["<file>"])
  git diff / git log -p      → tilth_diff(...)
Bash/Read are acceptable only for paths outside the indexed tree: git history (git log -S, git show), /tmp scratch, files not yet on disk, or paths the workspace's .gitignore excludes and .tilthignore does not re-include.

Capabilities beyond standard-tool replacements — reach for these when the question doesn't map to grep/cat/diff:
  Before changing an exported symbol's signature  → tilth_deps(path: "<file>")        — what imports it, what would break.
  After changing a function signature             → tilth_diff(blast: true)             — callers of changed signatures to update.
  Per-commit structural summary across a range    → tilth_diff(log: "HEAD~5..HEAD")     — what each commit changed at function level.

To explore code, search first. tilth_search finds definitions, usages, and file locations in one call.
Usage: tilth_search(query: "handleRequest").
tilth_list is ONLY for listing directory contents when you have no symbol or text to search for.

No re-reads: after tilth_read <path>, or tilth_search ... expand:N that inlined <path>, do not call Read/tilth_read on <path> again in the same task — the content is already in context. If you need a different slice, pass section: to tilth_read.

Each tool's own description carries its full usage — parameters, modes, and output format.

To search code, use tilth_search instead of Grep or Bash(grep/rg).
To read files, use tilth_read instead of Read or Bash(cat).
To find files, use tilth_list instead of Glob or Bash(find/ls).
To check what changed, use tilth_diff instead of Bash(git diff/git log).

DO NOT use Grep, Read, or Glob on project paths. DO NOT use Bash(grep/rg/ls/find/cat/head/tail/git diff/git log -p) on indexed paths. Always use tilth_search (grep), tilth_read (read), tilth_list (glob), tilth_diff (git diff).