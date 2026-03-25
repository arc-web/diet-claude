# lean-ctx: One-Shot Claude Code Implementation

This file contains a complete, copy-paste implementation prompt for setting up lean-ctx as an MCP server in Claude Code. It was validated against lean-ctx v1.8.2 on macOS and includes corrections for undocumented behaviors discovered during testing.

## How to Use

**For humans:** Copy the prompt below and paste it into Claude Code. It will install, configure, test, and report in one shot.

**For agents:** If you are an AI agent reading this repo, execute the prompt below step by step. It is self-contained and includes all the commands you need.

---

## Implementation Prompt

```
Implement lean-ctx (https://github.com/yvgude/lean-ctx) as an MCP server for Claude Code. Use the actual repo documentation to guide each step. Do not guess at commands; reference only what lean-ctx actually supports.

## PHASE 1: INSTALL

1. Check if Rust is installed: `rustc --version && cargo --version`
   - If missing: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y && source $HOME/.cargo/env`

2. Install lean-ctx via cargo: `cargo install lean-ctx`
   - If that fails, try building from source:
     git clone https://github.com/yvgude/lean-ctx.git
     cd lean-ctx/rust
     cargo build --release
     cp target/release/lean-ctx ~/.local/bin/

3. Verify: `lean-ctx --version` — must return 1.4.1 or higher.

## PHASE 2: CONFIGURE

4. Register MCP server with Claude Code using the full binary path to avoid PATH issues:
   `claude mcp add lean-ctx $(which lean-ctx)`

5. Install the 23 shell aliases globally:
   `lean-ctx init --global`
   Then source your shell profile (`source ~/.zshrc` or `source ~/.bashrc`).

6. Set CRP mode to compact (not tdd) to avoid symbol compression degrading output quality:
   `lean-ctx config init` — creates ~/.lean-ctx/config.toml if it doesn't exist.
   Then edit ~/.lean-ctx/config.toml and set crp_mode to "compact".
   Also export the env var as a fallback: `export LEAN_CTX_CRP_MODE=compact`
   Source your shell profile again after adding the export.

## PHASE 3: FUNCTIONAL TESTS — SHELL HOOK

IMPORTANT: The -c flag requires unquoted arguments. Use `lean-ctx -c git status` NOT `lean-ctx -c "git status"`. The README shows quoted syntax but the binary expects unquoted args. The shell aliases handle this correctly by design.

Run each test and capture BOTH the raw output and compressed output so we can verify real savings.

7. Test A — Git compression:
   Run `command git status` (raw, bypasses alias) and count the output lines.
   Then run `lean-ctx -c git status` and count the output lines.
   Report both line counts and whether the compressed version includes a savings line like [lean-ctx: X→Y tok, -Z%].

8. Test B — Directory listing compression:
   Run `command ls -la` raw.
   Then run `lean-ctx -c ls -la .` compressed.
   Report both outputs and confirm the compressed version shows token savings.

9. Test C — Shell hook aliases are active:
   Run `type git` and confirm it shows git is aliased to 'lean-ctx -c git' or similar.
   Run `type curl` and confirm the same pattern.
   If aliases aren't active, re-source the shell profile and retry.

## PHASE 4: FUNCTIONAL TESTS — MCP TOOLS

These are the 8 MCP tools that editors use. We need to verify they work via CLI since Claude Code will call them through the MCP protocol.

For Tests D-F, pick a REAL source file in the current project with at least 50+ lines of code. Do not use trivially small files; the cache and compression savings are meaningless on tiny files.

10. Test D — ctx_read (full mode with caching):
    Run: `lean-ctx read <file>`
    Then immediately run it again on the same file.
    The second read should show dramatically fewer tokens (near ~13) because of MD5 session cache.
    Report the token count from both reads. If the file is too small to show meaningful savings, pick a larger one and re-run.

11. Test E — ctx_read (map mode):
    Run: `lean-ctx read <same_file> -m map`
    Confirm it outputs a dependency graph + API signatures format (not the full file content).
    Report the token savings percentage shown.

12. Test F — ctx_read (signatures mode):
    Run: `lean-ctx read <same_file> -m signatures`
    Confirm it shows function/class signatures only.

13. Test G — ctx_tree:
    Run: `lean-ctx ls .`
    Confirm it produces a compact directory listing with file sizes and dir/file counts.

14. Test H — ctx_search:
    Run: `lean-ctx grep "fn " src/` (or any pattern relevant to the project).
    Confirm results are grouped and compressed.

15. Test I — ctx_compress (context checkpoint):
    Run: `lean-ctx -c echo "This is a test of context compression for long conversation checkpointing"`
    Confirm the tool processes and compresses the output.

16. Test J — Token savings dashboard:
    Run: `lean-ctx gain`
    This should now show real data from all tests above. Report the full output including the per-command breakdown table and total savings.

## PHASE 5: VERIFY MCP REGISTRATION

17. Run: `claude mcp list`
    Confirm lean-ctx appears in the output as connected with the full binary path.

18. Run: `lean-ctx discover`
    This analyzes your shell history and reports commands that could benefit from lean-ctx compression but aren't currently using it. Report the output; this tells us what additional savings are available.

19. Run: `lean-ctx session`
    This shows adoption statistics. Report the output.

## PHASE 6: MAXIMIZE SAVINGS (POST-INSTALL TUNING)

20. If `lean-ctx discover` found uncompressed commands, add aliases for any high-frequency ones not covered by the default 23.

21. Run `lean-ctx dashboard` to start the web dashboard on localhost:3333. Open it and confirm the 5 KPI cards render (tokens saved, savings rate, commands, days active, cost saved). Report whether it loaded successfully. Then kill the dashboard process.

22. Confirm the config file is correct:
    Run: `lean-ctx config`
    Report the full output so we can verify crp_mode=compact and all paths are correct.

## FINAL REPORT

Produce this exact structure, filling in real values from every test:

LEAN-CTX IMPLEMENTATION REPORT
================================
lean-ctx version: [from step 3]
Install method: [cargo | source]
MCP registered in Claude Code: [yes/no + path shown, from step 17]
Shell profile modified: [exact path from step 5]
CRP mode: [value from lean-ctx config output]
Aliases active: [yes/no, with proof from step 9]

SHELL HOOK TESTS:
  Test A (git status): raw=[X lines] compressed=[Y lines] savings_shown=[yes/no, Z%]
  Test B (ls): raw=[X lines] compressed=[Y lines] savings_shown=[yes/no, Z%]
  Test C (aliases): git=[alias output] curl=[alias output]

MCP TOOL TESTS:
  Test D (ctx_read full, cached): first_read=[X tok] second_read=[Y tok] file_used=[path, line count]
  Test E (ctx_read map): savings=[Z%] output_format=[correct/incorrect]
  Test F (ctx_read signatures): output_format=[correct/incorrect]
  Test G (ctx_tree): format=[correct/incorrect] savings=[Z%]
  Test H (ctx_search): format=[correct/incorrect]
  Test I (ctx_compress): [working/not working]

DASHBOARD (lean-ctx gain):
  [paste full output from step 16]

ADOPTION & DISCOVERY:
  lean-ctx discover: [paste output or summary from step 18]
  lean-ctx session: [paste output from step 19]
  lean-ctx config: [paste output from step 22]
  Web dashboard: [loaded successfully / failed]

claude mcp list:
  [paste full output from step 17]

Errors encountered: [list or "none"]
Recommended additional aliases: [any from discover output, or "none needed"]
================================
```

---

## Known Gotchas (from real testing)

1. **`-c` flag quoting**: Use `lean-ctx -c git status` (unquoted args), not `lean-ctx -c "git status"`. The binary treats a quoted string as a single command name and fails with `command not found`.

2. **MCP PATH issues**: `claude mcp add lean-ctx lean-ctx` may fail if `~/.cargo/bin` isn't in Claude Code's PATH. Always use the full path: `claude mcp add lean-ctx $(which lean-ctx)` or `claude mcp add lean-ctx /path/to/.cargo/bin/lean-ctx`.

3. **Small test files**: Cache and compression savings are meaningless on 1-3 line files. Use real source files with 50+ lines for meaningful test results.

4. **Shell profile**: `lean-ctx init --global` modifies `~/.zshrc` (macOS) or `~/.bashrc` (Linux). You must source it or restart your shell for aliases to take effect.
