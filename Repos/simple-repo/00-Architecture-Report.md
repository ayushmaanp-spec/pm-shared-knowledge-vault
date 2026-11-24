# Graph Report - simple-repo  (2026-11-24)

## Corpus Check
- cluster-only mode — file stats not available

## Summary
- 26 nodes · 73 edges · 4 communities (3 shown, 1 thin omitted)
- Extraction: 89% EXTRACTED · 11% INFERRED · 0% AMBIGUOUS · INFERRED: 8 edges (avg confidence: 0.85)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `75bcc558`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- Community 0
- Community 1
- Community 2
- Community 3

## God Nodes (most connected - your core abstractions)
1. `system_excu()` - 9 edges
2. `ingreen()` - 8 edges
3. `next_repo()` - 8 edges
4. `repo_status()` - 7 edges
5. `repo_branch()` - 7 edges
6. `repo_checkout()` - 6 edges
7. `repo_forall()` - 6 edges
8. `repo_push()` - 6 edges
9. `repo_remote()` - 6 edges
10. `repo_sync()` - 6 edges

## Surprising Connections (you probably didn't know these)
- `pretty_branchinfo_output()` --calls--> `ingreen()`  [EXTRACTED]
  repo_utils.py → repo_utils.py  _Bridges community 0 → community 2_
- `repo_status()` --calls--> `ingreen()`  [EXTRACTED]
  repo_utils.py → repo_utils.py  _Bridges community 0 → community 1_
- `repo_checkout()` --calls--> `system_excu()`  [EXTRACTED]
  repo_utils.py → repo_utils.py  _Bridges community 0 → community 3_
- `repo_status()` --calls--> `inred()`  [EXTRACTED]
  repo_utils.py → repo_utils.py  _Bridges community 1 → community 3_
- `repo_branch()` --calls--> `subprocess`  [EXTRACTED]
  repo_utils.py →   _Bridges community 2 → community 1_

## Import Cycles
- None detected.

## Communities (4 total, 1 thin omitted)

### Community 0 - "Community 0"
Cohesion: 0.54
Nodes (7): ingreen(), next_repo(), repo_checkout(), repo_forall(), repo_push(), repo_remote(), repo_sync()

### Community 2 - "Community 2"
Cohesion: 0.43
Nodes (4): BranchInfo, pretty_branchinfo_output(), repo_branch(), update_branch_info()

### Community 3 - "Community 3"
Cohesion: 1.00
Nodes (3): inred(), repo_init(), system_excu()

## Knowledge Gaps
- **1 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `repo_branch()` connect `Community 2` to `Community 0`, `Community 1`?**
  _High betweenness centrality (0.033) - this node is a cross-community bridge._
- **Why does `repo_status()` connect `Community 1` to `Community 0`, `Community 2`, `Community 3`?**
  _High betweenness centrality (0.021) - this node is a cross-community bridge._