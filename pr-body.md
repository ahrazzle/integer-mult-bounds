## Change and mathematical scope

This PR adds the research package `research/cubic-odd-s-shift-floor`: scoped proofs, a complete finite classification and exact certificates. **No κ, supplier or selected-result change.**

**Theorem A: integer shifts need four queries at 82 odd-S arities.** In PR6653's integer-shift alphabet, the only source change is `a += k*b` with integer `k`, and the only query is `y += q*a^3` with dyadic `q`. Let `S = r(r-1)/2` be odd.

- A mod-8 lemma, in the shifted coordinates `w = 6c-(2r-1)` that PR6653 suggested, shows that exactly one node has the other parity.
- That node's weight is a nonzero integer, which forces the smaller gap below `2S`.
- The node equation factors as `(w_0+3x)(w_0+x+4y) = 3x^2-M`.

So for odd S the question is a finite computation. Over the 99 odd-S arities `3 <= r <= 200`:

- exactly 17 admit three queries, with 34 node sets in total, and every one is already covered by PR6653's classes or certificates;
- the other 82, including **15, 22, 30, 31**, need exactly four, and a four-query certificate is given for each.

**Theorem B: twenty new three-query arities.** At 48, 49, 57, 60, 69, 84, 88, 96, 105, 112, 120, 121, 124, 132, 133, 148, 169, 177, 184 and 192, the package gives exact dyadic certificates in the PR6212/PR7475 general-frame contract. Each comes with a literal integer-shear program that restores both arbitrary sources. With PR6212's lower bound, three queries is the exact minimum at all twenty. At five of these arities (84, 105, 121, 132, 148), two of the three rows are non-monic.

**Remaining obligations.**
- Theorem A is about the integer-shift alphabet only. 44 of the 82 arities already have three-query repairs that read `b`.
- 38 odd-S arities and the even-S arities 40, 61, 141, 165, 168, 193 and 196 remain open in general frames.
- Shear counts are upper bounds.
- Physical prices are unknown.

## Task and overlap

- **Base:** `e54523517d0f03e58352e647c403a5af3e525a57`, which is current main.
- **Task card:** `joint-query-controller`, scope **changed**. This is the same coefficient/restoration side family as #7475, #7478 and #7479. TASK.json gives the reason and the connection to the bottleneck.
- **What it answers:** it continues PR6653's Open next question 1 and takes up PR7479's Open next question 2 (restricted families).
- **Relation to #7539 (same team).** mithrax-exo's #7539 states the same two results: the same 17/82 split at odd S and the same twenty general-frame rows. This PR is meant to replace it. What it adds:
  - explicit checks that still run under `python -O`;
  - a separately written second checker that takes the other divisor route;
  - pinned predecessor sources and the registered replay;
  - a four-query certificate at each of the 82 arities, so four is exact rather than only a lower bound;
  - executed programs that restore both sources.

  My review on #7539 lists the differences in detail. If both PRs are kept, review only one of them.
- **Overlap checks:**
  - I fetched all open PR heads from #7480 to #7538, plus the ledger's cubic PRs (#6200–#6797), #7475, #7478 and #7479.
  - The nearest concurrent work is Nick Petalas's #7506. It shows that at `r = 15` any frame needs two rows of height above 64. Theorem A is complementary: it covers integer shifts of every height, at 82 arities.
  - `find_overlap.py --offline` found only the local cubic packages already cited.
  - These checks are leads, not a novelty certificate.

## Validation

CPython 3.12, standard library only:

```sh
python3.12 -B scripts/research_package.py --package research/cubic-odd-s-shift-floor
```

The registered replay has five commands: `verify.py` and `independent_check.py`, each run normally and under `-O`, plus `build.py --check`. All five pass. The expected markers are `PASS_CUBIC_ODD_S_SHIFT_FLOOR` and `PASS_CUBIC_ODD_S_SHIFT_FLOOR_INDEPENDENT`.

**`verify.py`**
- Authenticates four pinned predecessor files, checks four textual anchors and PR6653's uncovered-arity list, and rejects two byte mutations.
- Runs the complete search for Theorem A. It factors `3x^2-M` over every odd gap `|x| < 2S` with a quadratic sieve and enumerates the divisor route.
- Checks every certificate by exact coefficient comparison and recomputes the weights by elimination.
- Executes every program symbolically over all `r` retained children, and on dirty rational inputs.
- Runs nine rejection controls. Among them, the same search at `r = 12` misses PR6653's same-parity certificate, which shows that the parity lemma is load-bearing.

**`independent_check.py`**
- Imports neither of the other scripts.
- Takes the other route of the proof: it enumerates the larger gap over the divisors of `S·N`, uses Cipolla square roots and solves the node quadratic.
- Checks identities at six non-proportional points and runs programs in a separate interpreter.
- Rejects five mutations.

**`build.py --check`** regenerates `CERTIFICATES.json` byte for byte.

**Publication preflight.** `scripts/check_research_commit.py --check-modes --check-source-history` passes on a full-history clone (13 files, source pin `e545235`).

**What is proof and what is search.** Theorem A's search is the proof: Lemmas 3–5 make it complete, and both checkers run it in full. The twenty general-frame rows came from an external search that is not included; they are the same rows as in #7539. They are accepted only by the exact checks. The two checkers are separate AI-assisted implementations; neither human nor proof-kernel review is claimed.

## Credit

- **Nick Petalas:** the PR6200 reader, defect and comparison; the PR6653 node lemma, integer-shift alphabet, shifted coordinates and open question.
- **apoursh:** PR6212 (the dyadic lower bound and general-frame contract), PR6215, PR7475, PR7478 and PR7479.
- **AAAmirinu** (PR6246) and **CubeJerry** (PR6250).

Prepared with substantial AI assistance from several agents, including Anthropic Claude, as disclosed in TASK.json. Apache-2.0.

## Open next questions

These are recorded in the package README and TASK.json.

1. **General frames at the 38 open odd-S arities, starting with r = 15.**
   - *First step:* extend the parity lemma to rows with even `u`, which correspond to dyadic shifts, and combine it with #7506.
   - *Blocker:* with non-monic rows there is no integral weight, so the size bound is lost.
2. **The even-S arities 40, 61, 141, 165, 168, 193 and 196.**
   - *Local evidence (not packaged):* a search with two integer nodes `|c| <= 12000` and the third node forced found nothing.
   - *First step:* search with two non-monic rows.
3. **Prices.** Price the four-shear programs at 124, 169, 177 and 184 against `r` eager shears, using #7475's event inequality.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_01ErDAeESzficxtkpvZpvqVP
