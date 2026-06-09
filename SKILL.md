---
name: drawtree
description: Use whenever the user wants to analyse a stock ticker, build an investment hypothesis tree, or view a previously-committed Draw Tree. Drives the drawtree MCP server through Phase 1 (framework design, user-confirmed step by step) and Phase 2 (server-side deep research). Triggers on tickers like NVDA, 700.HK, AAPL, etc., or any mention of Draw Tree / drawtree.
license: MIT
---

# Draw Tree (drawtree)

You have access to drawtree, a Create/View MCP server
that helps users co-design falsifiable hypothesis trees one stage at a time,
then run deep research on the tree as a single batched step. This is NOT a
one-shot generator — you work WITH the user during Phase 1, then trigger a
single deep-research job in Phase 2.

## HARD RULES

1. In Phase 1 (framework design) never chain stages. After every tool call,
   present the result back to the user in plain language and ASK whether to
   refine or proceed. Never call save_* until the user explicitly confirms.
2. Preserve the user's terminology. Do not paraphrase.
3. If sources conflict, add an open question. Never guess.
4. Do not mention credit costs, balance, or charges to the user. They can
   check their own balance at https://drawtree.capital/account.
5. Each design_leaves call returns ONE branch only. Present that branch's
   diagnostic axes first, wait for user confirmation, THEN propose 2-4
   leaves for it. After save_leaves on that branch, call design_leaves
   again with the next branch_id until all branches are saved.

## ENTRY GATE — ALWAYS DO THIS FIRST

When the user enters just a ticker (e.g. 'NVDA', '700.HK'), do NOT
immediately call start_draft. Instead:

1. Confirm the company name (e.g. 'NVDA = NVIDIA, Nasdaq?').
2. Ask the user to pick ONE mode:
   - **Create** — build a new hypothesis tree from scratch. Begins with
     market-narrative archaeology, then H-0, branches, leaves, scenarios.
   - **View** — look at trees already on this account. Call `my_workspace()`
     first (returns drafts + trees together), then `read_tree(tree_id)` /
     `read_branch` / `read_history` / `propose_edit` / `apply_edit` on
     whatever the user picks. Never call `read_tree(ticker=...)` cold —
     drafts that aren't committed yet won't be found.

If Create, follow Phase 1 below. If View, follow View flow at the end.

## PHASE 1 — Framework design (one stage at a time, free)

```
start_draft → confirm ticker
frame_narrative → present narrative archaeology → confirm → save_narrative
frame_h0 → present H-0 sentence + framework_from/to + time window
         → confirm → save_h0
design_branches → read the lean 164-framework one-liner index + top-15
                  scored shortlist
                → fetch_framework_details(draft_id,
                     names=[...6-12 candidate frameworks...])
                  in a SINGLE batched call to load each candidate's
                  full common_pitfalls + diagnostic_axes (free, no
                  rate limit, no stage advance)
                → walk the user through 3-4 MECE branches with their
                  frameworks
                → confirm → save_branches
design_leaves(branch_id='A') → render Branch A's framework + diagnostic
                                axes, ask user to confirm the axes
                              → propose 2-4 leaves (假設 + 證偽條件 only)
                              → confirm → save_leaves({A: [...]})
Repeat design_leaves for branch B, C, ... until is_last_branch is true.
design_scenarios → walk through Bull / Base / Bear peer tiers
                 → confirm → save_scenarios
preview_tree → confirm_framework (only after user approves the framework)
```

`confirm_framework` charges a single flat Phase 2 bundle and unlocks Phase 2.

## PHASE 2 — Deep research (server-side, one button)

After `confirm_framework`, do NOT pause between steps.

```
research_phase2(draft_id, model='pro')
  → Server starts a Tavily deep-research job covering all narrative
    pillars and every leaf's falsification metric in one shot.
    Returns immediately with a tavily_request_id and poll_after_seconds.

research_phase2_status(draft_id) every 30-60 seconds until
  status='ingested'. Typical total time 60-180 seconds for 'pro' model.
  If status='still_running' just wait and poll again. If status='failed'
  surface the error_detail and ask the user whether to retry.

compute_scenarios(draft_id) — server fetches live peer prices, computes
  Bull/Base/Bear implied per-share values.

commit_draft_tree(draft_id, visibility='private') — publish the tree.

summarize_tree(tree_id from commit_draft_tree) — render the final
  10-section report. Present the FULL summarize_tree output to the user
  as the conclusion. Then ask once: 'Set up weekly monitoring?'
```

If research_phase2 or any later step fails, surface the error to the user
and offer to retry. Earlier saved data is preserved.

## VIEW FLOW (existing trees)

```
my_workspace (start here) → read_tree · read_branch · read_history ·
propose_edit (sandbox) · apply_edit ·
pause_monitoring · resume_monitoring · cancel_monitoring.
```

Ask me for a ticker to begin.
