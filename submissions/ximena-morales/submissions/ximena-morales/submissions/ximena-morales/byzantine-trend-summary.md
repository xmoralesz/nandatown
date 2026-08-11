# NANDA Town — byzantine_agents trend: 0% -> 10% -> 20% -> 30%

Same seed (42), same 100 agents, same marketplace scenario throughout. Only
`failures.byzantine_agents` changes. Absolute counts below come from parsing
`sold:` and `buy:` send events directly (see prior turns for methodology);
the 0.30 run's raw counts were captured in this turn, 0.10/0.20 in earlier
turns, baseline (0%) in the first run.

| byzantine_agents | buy: requests | sold: completions | deal_rate | message_count | unique_pairs | unanswered buy requests |
|---|---|---|---|---|---|---|
| 0.00 (baseline) | 500 | 266 | 0.5320 | 2000 | 467 | 0 |
| 0.10 | 273 | 166 | 0.6081 | 1086 | 257 | 3 |
| 0.20 | 171 | 103 | 0.6023 | 666 | 161 | 9 |
| 0.30 | 125 | 78 | 0.6240 | 474 | 118 | 13 |

## Does the trend keep scaling?

**Absolute volume collapse: yes, monotonic and roughly exponential decay.**
`buy:` and `sold:` counts fall at every step (500->273->171->125 and
266->166->103->78). The decay rate per +10 points of byzantine_agents
shrinks each step (buy: -45.4%, -37.4%, -26.9%), consistent with a
saturating effect: once the "easy" byzantine-touched conversations are
gone, each additional 10% of corrupted agents has fewer clean pairs left
to spoil.

**deal_rate: mostly up, but not cleanly monotonic.** 0.5320 -> 0.6081 ->
0.6023 -> 0.6240. It dipped slightly from 10% to 20% before rising again
at 30%. It remains a ratio artifact (denominator shrinking faster than
numerator overall), not evidence of a healthier market — confirmed by the
absolute collapse above.

**marketplace_all_responded failures: yes, scales up roughly linearly.**
Unanswered buy requests: 0 -> 3 -> 9 -> 13. This is the most reliable
scaling signal — it tracks the byzantine fraction directly instead of
being distorted by a shrinking denominator.

**marketplace_no_double_sell and marketplace_price_agreement: still PASS
at every level tested (0%, 10%, 20%, 30%).** Byzantine agents corrupt
messages and starve responses, but do not (at these levels, with this
scenario) cause a seller to double-sell or a sale to settle at a price
that was never offered.
