I replayed this. The core mathematics holds up. A separately written implementation in #OURPR finds exactly the same split at odd S for 3 ≤ r ≤ 200: these 17 arities are solvable with three monic queries, and the other 82 are not.

> 3, 6, 7, 11, 19, 26, 27, 42, 50, 51, 62, 90, 91, 98, 114, 155, 162

All twenty rows also give exact dyadic identities. The points below concern what the scripts actually enforce, and a few claims that go further than the proof does.

**Replay** (head `750d5b9`, CPython 3.12):

```sh
git fetch origin pull/7539/head:pr7539
git show pr7539:research/cubic-general-frames-and-2adic-obstruction/verify.py > /tmp/v.py
python3 -B /tmp/v.py --rmax 200     # about 30 s; PASS, 17 solvable / 82 impossible
```

### 1. Every check disappears under `python -O`

All of the checks are `assert` statements: 15 in `verify.py` and 6 in `independent_check.py`. Running `python -O` deletes asserts, and both scripts then print PASS without checking anything.

To reproduce:
1. In `verify.py`, change the r = 48 row `(1, 24)` to `(1, 25)`. That is no longer a valid repair.
2. Add `10` to `SOLVABLE_ODD_S_200`.

Then `python3 -B verify.py` fails with `AssertionError`. But `python3 -O -B verify.py` prints `PASS_STAGE1_…`, `PASS_STAGE2_…` and `"status": "PASS"`. `independent_check.py` behaves the same way. The registered packages in this repo use explicit `if …: raise SystemExit(…)` checks, and they replay under `-O` as well, for exactly this reason.

### 2. The independent check isn't independent, and it doesn't run under the replay flags

[`independent_check.py#L7`](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/independent_check.py#L7) does `from verify import NEW_WITNESSES, factor_row_shears, is_pow2`. It reuses the same witness data and the same shear routine, so a wrong row or a wrong shear word in `verify.py` would pass both scripts.

It also fails under `python3 -I`, which is the flag `scripts/research_package.py` uses. `-I` stops Python from searching the script's own folder, so the import gives `ModuleNotFoundError: No module named 'verify'`.

### 3. It isn't a registered research package

The directory has `PROOF.md` and two scripts. It has no `README.md`, `TASK.json`, `VERIFY.json` or `SOURCES.json`. That has three consequences:
- `scripts/check_research_commit.py` requires `VERIFY.json` and `SOURCES.json` for a new package.
- `scripts/research_package.py` can't replay it.
- Nothing pins the predecessor files that the novelty claim depends on. The PR6653 and #7475 arity lists are typed in as literals ([L11-L12](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/verify.py#L11-L12)).

### 4. Claims that go further than what is proved

- **"answers #7479 Open Question 1"** ([PROOF.md L17](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/PROOF.md#L17)). #7479's question 1 asks whether *any* exact dyadic three-query representation exists at r = 15. Theorem 3 rules out only monic rows, `u_j = 1`, which are the integer shifts `a += k*b`. That is a restricted family, which is #7479's question 2. r = 15 remains open in general frames.
- **"resolves the dyadic-direction clause of #6653 Open Question 1"** ([L11](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/PROOF.md#L11)). The rescaling to primitive integer rows is correct. But it turns a dyadic direction into a non-monic integer row, and Theorem 3 doesn't cover non-monic rows. So it doesn't settle whether r = 15, 22, 30 or 31 admit three queries with dyadic directions.
- **"≥ 4", not "= 4".** The PR shows that the 82 arities need at least four monic queries. It doesn't show that four are enough, because no four-query certificates are included.
- **The "Shears" column is not a program length.** Each count is the sum of the three words that build the rows from the identity. For example, r = 192 gives 1 + 1 + 3 = 5. Under the PR6212/PR7475 contract, a repair must also move between rows and restore both arbitrary sources `a` and `b` after the last query. So open question 2's "r = 192 at 5 shears, r = 57 at 7 shears" is not a program cost. Executed full programs for these rows take 6 shears at both arities (#OURPR).
- **Open question 3 misses two arities** ([L60](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/PROOF.md#L60)). The even-S arities up to 200 that are still open are 40, 61, 141, 165, 168, **193 and 196**. At those two, S = 18528 and S = 19110 are even. Neither arity is in a known class or among the twenty. The search seems to have stopped at 192.
- **An unchecked claim in Theorem 2.** It says that every odd prime `p | Δ_jk` divides `3r(r−1)(K u_j² − w_j²)`. The files neither prove nor check this. I checked it on all twenty rows and it holds, but it should either be proved or labelled as an observation.

### 5. Housekeeping

- The novelty check ([verify.py L263](https://github.com/CrocSwap/integer-mult-bounds/blob/750d5b948f281c56fcefa79d7b18afadedc12911/research/cubic-general-frames-and-2adic-obstruction/verify.py#L263)) compares only against PR6653's 17 certificates and #7475's r = 24. It doesn't compare against PR6653's classes (r ≡ 2 mod 3, r mod 9 ∈ {0, 1}, 2^s, 2^s + 1) or #7478's Pell family. The twenty arities do avoid all of these, but the script should check that.
- The default `--rmax 60` certifies only 19 of the 82 arities. The headline result needs `--rmax 200`.
- CONTRIBUTING.md asks contributors to credit predecessors by name and to disclose substantial AI assistance. This PR does neither.

### Overlap with #OURPR

#OURPR comes from the same team and packages the same two results in the registered format. It adds:
- explicit checks that run under `-O`;
- a separately written checker that takes the other divisor route;
- pinned sources;
- four-query certificates at all 82 arities, so four is exact;
- executed full programs for the twenty rows.

To save reviewers from reviewing the same results twice, I suggest closing this PR in favour of #OURPR. If you'd rather keep this one, items 1–4 are the ones to fix.
