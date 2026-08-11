# Byzantine agents in the marketplace: does corruption break integrity, or something else?

**Scenario:** `marketplace` (Tier 1, state-machine agents, seed 42)
**Setting changed:** `failures.byzantine_agents` — one knob, swept `0.0 → 0.10 → 0.20 → 0.30`, everything else identical.

## Why this setting

I'm building agentic booking into a travel platform: once a group vote closes, an agent books the chosen experience, flight, or hotel on the user's card. A byzantine seller is a good model of a *provider that returns corrupt or missing responses* to that booking request — the concrete risk when an agent transacts on someone's card. So I stressed the marketplace with a rising fraction of such providers and watched the payment-integrity invariants.

## Hypothesis (written before running)

- Completed deals would drop, because agents fail to agree.
- Double-sells would increase (same product sold to two buyers).
- `marketplace_price_agreement` would FAIL, as bad actors sell at terms other than agreed, to exploit.
- Responses would stay near 100%, because a malicious seller still has an incentive to reply.

## Results

| byzantine_agents | buy | sold | deal_rate | messages | unique pairs | unanswered |
|---|---|---|---|---|---|---|
| 0.00 (baseline) | 500 | 266 | 53.2% | 2000 | 467 | 0 |
| 0.10 | 273 | 166 | 60.8% | 1086 | 257 | 3 |
| 0.20 | 171 | 103 | 60.2% | 666 | 161 | 9 |
| 0.30 | 125 | 78 | 62.4% | 474 | 118 | 13 |

Validators at every level: `marketplace_no_double_sell` PASS, `marketplace_price_agreement` PASS, `marketplace_all_responded` **FAIL** (3 / 9 / 13 requests unanswered).

## What I found

1. **Integrity held; responsiveness broke.** No double-sells and no price mismatches at any level. The only broken invariant was `all_responded` — buy requests that received neither a sale nor a rejection.
2. **Absolute business collapsed, even though the rate rose.** `deal_rate` went *up* (53% → 62%), which looked like a healthier market. It wasn't: absolute sales fell 266 → 78 and buy requests fell 500 → 125. The ratio rose only because the denominator shrank faster than the numerator.
3. **Decelerating damage (saturation).** Volume fell fastest at first (−45%, −37%, −27% in buy requests). Once the easily-corrupted pairs are gone, each added 10% of byzantine agents finds fewer clean pairs left to disrupt.
4. **The one trustworthy signal was a raw count.** Unanswered requests (0 → 3 → 9 → 13) scaled almost linearly and, unlike any ratio, could not be distorted by a shrinking denominator.

## Investigation

The rising `deal_rate` contradicted my hypothesis and looked too good, so I did not trust it. I had Claude Code count *absolute* sales and buy requests per level (not just the rate) and re-derived `deal_rate = sold / buy` to confirm the formula. That confirmed volume was collapsing and the rate was an arithmetic artifact. Investigating why my fraud predictions failed, I concluded the simulator's byzantine agent is *non-strategic corruption* (garbled or silent), not a rational cheater optimizing for gain — which is exactly why integrity survived and only responsiveness fell.

## What I learned

"Malicious" in a distributed system does not have to mean "con artist." Here the risk was not being robbed — it was being left hanging. And a conversion-style rate can rise while the real business underneath it collapses, so under adversarial conditions I would trust direct failure counts over ratios. Both lessons transfer straight to payments.

## Tools and help (AI disclosure)

Claude Code drove the technical work: installing `nest-core`, running the baseline and the sweep, generating HTML reports, and computing absolute counts. Validation used `validate_trace()` in `nest_core.validators` (there is no `nest validators` CLI subcommand; Claude Code located the logic in the Python module). The repo docs `quickstart.md` and `writing-a-scenario.md` guided the YAML syntax. I set the direction, wrote the hypothesis before running, and drove the investigation into absolute volume when `deal_rate` looked suspicious. I worked with Claude Code in Spanish; all committed file content is in English.

## Files in this PR

- `byzantine.yaml` — the edited scenario (`marketplace` + `byzantine_agents`)
- `byzantine-trend-summary.md` — the four-level comparison table and analysis
- HTML reports and validator screenshots are attached to the application form
