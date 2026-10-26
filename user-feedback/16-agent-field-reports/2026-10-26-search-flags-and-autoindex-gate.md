# Reproduced the search-flag and autoindex performance problems on rc.99 (2026-10-26)

**Agent:** Claude Code (Haiku 5.5)  ·  **XERJ:** xerj v1.0.0-rc.99 (built from source)  ·  **Platform:** Linux x86_64 (WSL2)

**Pointed at:** the XERJ repository itself, plus the release gate's 1 GB synthetic corpus (44,758 files, seed 2421).

**Used it for:** autoindex and search, reproducing two open issues, and a small CLI fix.

**Verdict:** The unknown-flag path was the worst part. `xerj search --index foo` printed "unexpected argument" followed by the whole usage block, so it read like help and did not say which flag was wrong. Autoindex on the 1 GB corpus, built from main, timed out at 930 s with 3,837 indices and 33 s search p95, which is the #2422 symptom. The release gate also reported "node never became healthy" behind an HTTP proxy, even though the node was up. Setting `NO_PROXY` for loopback fixed it. The `--help` text and the error for a second plain word were clear.

**Numbers:**
- `scripts/perf/autoindex_release_gate.sh --tier 1gb` on main (e97ccbde8): 3,837 indices, search p95 33.6 s, autoindex timed out at 266 of 1,024 MB.
- The same gate with PR #2430 applied: 33 indices, search p95 26.2 s, timed out at 214 of 1,024 MB.
- Not measured: a 5 GB corpus, and repeated runs.

**Filed alongside:** #2458 (unknown-flag refusal; the fix is on a branch pushed to a fork, PR not yet opened). The proxy problem belongs in #2433, not yet filed.
