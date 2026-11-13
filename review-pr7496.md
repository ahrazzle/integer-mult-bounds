Thanks for this. A stock-Lean (`Init`-only) check of the #7000 arithmetic is a reasonable idea, and the file does run through `lean` without errors. I replayed it and found seven things to fix before it's ready. They're listed below in order of importance, each with the exact lines and how to reproduce it.

**Replay.** Run with `leanprover/lean4:v4.21.0`, the repo's pinned toolchain, on head `3b994da`:

```sh
git fetch origin pull/7496/head:pr7496
git show pr7496:formal/lean/Frontier7000ZeroDeps.lean > /tmp/F.lean
lean /tmp/F.lean      # about 0.4 s, no errors
```
```
'…frontier7000_kappa_hierarchy_and_supplier_margin' does not depend on any axioms
'…frontier7000_unit_job_and_rank_mass_ledger' does not depend on any axioms
'…rank_one_probe_det_zero' depends on axioms: [propext]
'…rank_one_fusion_impossible' depends on axioms: [propext]
'…role275_fixed_fusion_impossible' depends on axioms: [propext]
```

That is five reports. The file has fourteen `#print axioms` lines, and nine of them never run.

### 1. The new big-integer theorems from `2235b11` are commented out, and they wouldn't compile if they weren't

In Lean, `/- … -/` comments nest. The new commit pasted a second header, starting `/-!`, at [L4](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L4). That is inside the original doc comment, which opens at L1. The `-/` at L25 closes only the inner comment, so everything from L4 to L174 is comment text. That range includes:
- `pow2_add` and `pow2_mono`;
- `frontier7000_cover_119542_and_kstar11_bounds`;
- `frontier7000_interval_gate_141746_and_kstar14_bounds`;
- their `#print axioms` lines.

Lean checks only the second copy of the module, at L176–L254, which is the old file. That is why the replay prints five reports, not nine.

If the block is compiled on its own (`sed -n '4,161p'`), there are 10 errors:
- **[L63](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L63): type mismatch.** The copied `rank_one_probe_det_zero` says `congrArg (fun t => s1 * t) (Int.mul_comm b a)`. The working copy at L222 says `fun t => s1 * (s2 * t)`.
- **[L112–L114, L119–L120, L138–L139, L146–L147](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L112-L147): "maximum recursion depth has been reached".** This happens on every `decide` that involves a 2228th power.
- **Axioms.** As a result, `#print axioms` reports `sorryAx` for both new theorems and for the three determinant theorems.

So doc item 4, "proving `4096 * kstar < 2 ^ H` with zero axioms", is not established by this file.

The inequalities themselves are true. I checked them with exact integers in Python: `(H14+1)^2228` has 141,746 bits, and `4096·K*` has 14,085 bits at `H = 14174600000000000000`. But two problems remain:
- The H14 facts are already checked: by `docs/research/pr-intake-evidence/pr7000-admission/check_interface_bindings.py` (L120–L122) and stated in that folder's `PROOF-REVIEW.md` (L160–L161: "K* has 14073 bits, and 4096K* < 2^H").
- `H11 = 14174600000000000` and the "minimal fixed-point crossing `j = 141,746`" appear nowhere in the repo.

Also, [L21](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L21) says `(H14 - step14 + 1)^2228 >= 2^141745` "fails (`wins(141745) = false`)". The theorem at L132–L137 asserts that same inequality as true, and it is true.

### 2. The "zero axioms" claim is false for three of the five live theorems

[L173](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L173) says the file "declares zero axioms". The doc says "zero axioms across all five theorems" ([L10](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/docs/research/zero-deps-lean-frontier-and-batch-obstructions.md#L10)) and "does not depend on any axioms" ([L13](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/docs/research/zero-deps-lean-frontier-and-batch-obstructions.md#L13)). But the three determinant theorems use `propext`, which comes in through `rw`. `propext` is harmless, and `formal/lean/README.md` already accepts it. The text should still say what the axiom report says.

### 3. The ledger checks numbers typed into the file, not the #7000 record

[`frontier7000_unit_job_and_rank_mass_ledger`](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L199-L211) and the κ hierarchy theorem prove only that the integers written in the file add up and compare as stated. For example, they prove 4075 + … + 6 = 6310. Nothing connects those integers to `research/shared-wire-split-composition/FULL-RESULT.json` or `certificates/selected-result.json`. If the same wrong value were typed into both the Lean file and the doc, both would still pass.

The repo already has a mechanism for this. [`formal/lean/sources.py`](https://github.com/CrocSwap/integer-mult-bounds/blob/main/formal/lean/sources.py) is run by `tests/test_lean_sources.py`, and it fails when a Lean literal differs from its certificate value. It only scans `KappaCheck/*.lean`, so this file is outside it.

To fix this:
1. Move the module under `KappaCheck/`.
2. Add `sources.py` rows that bind each literal to its JSON key. For example, `native.paid` sums to 6310, `native.helpers` to 1080, and `invoice.S` is 1520.
3. Add a SOURCES.md entry.

### 4. Several totals don't appear in the source package

The base numbers match the package README and JSON:
- 6310 MOVE;
- 400 main fields;
- 1080 helpers;
- the 1125 / 1520 rank surplus;
- the 192 removed rank-one fields and paid jobs.

These values, however, appear nowhere in `research/shared-wire-split-composition/`:
- the weighted "rank mass" sums 10962, 7650 and 18612;
- the net totals 7390, 1288, 7198 and 18420.

The source also never defines 1288 = 400 + 1080 − 192. For each of these quantities, either cite where it is defined and say why it matters for the #7000 claim, or drop it. As written, these lines read as verified frontier facts, but they are new arithmetic with no stated meaning.

### 5. `rank_one_fusion_impossible` is proved for integers only, but Role 275 needs fractions

[L233–L241](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/formal/lean/Frontier7000ZeroDeps.lean#L233-L241) quantifies the unknown scalars and wire coefficients `s1 s2 a b` over `Int`. The Role 275 rows are `(1, −1/2)` and `(1, +1/2)`. Multiplying the target rows by 2 makes the targets integers, but the unknowns may still be fractions, for example `a = 1/2`. So the theorem doesn't cover the case that `role275_fixed_fusion_impossible` names. The statement that's needed holds over any field: if the determinant is nonzero, there is no rank-one factorization.

The lemma is also not the substance of #7493. That PR excludes both repair deadlines using the 898 live chart-3 rows and the delayed chart-312 analysis, and the 2×2 determinant of the target rows is one line of that argument. Please present the lemma as a supporting identity, not as "instantiating #7493's impossibility".

### 6. Nothing builds this file

The file isn't a `lean_lib` in `formal/lean/lakefile.toml`, and `KappaCheck` doesn't import it. So `lake build` and CI never compile it, which is how item 1 got through. It sits in a Mathlib project without using Mathlib. That is fine, but it should be registered, or the doc should say it is checked only by a manual `lean` run.

### 7. The batch CP-SAT section describes code that isn't in the PR

[Doc §2](https://github.com/CrocSwap/integer-mult-bounds/blob/3b994da8f7dff1f4c495f7adf2058bcc70163eef/docs/research/zero-deps-lean-frontier-and-batch-obstructions.md#L15-L17) describes a 0-1 CP-SAT model that emits batch certificates "verified in < 100ms". No model, certificate or timing is included. Please mark it as a proposal or remove it.

### Smaller points

- CONTRIBUTING.md asks contributors to disclose substantial AI assistance. Please add a disclosure line and state the affiliation consistently.
- The κ comparisons are correct: κ > 118·2⁻¹⁵, κ > the #6030 κ, the 22,242,183 margin below .00385248, and 826,790,496 < 10⁹. But they restate `selected-result.json`, so they add value only once they are bound to it, as in item 3.

### Suggested path

1. Delete the pasted block at L4–L161.
2. Keep the κ hierarchy and the ledger. Move them under `KappaCheck/` and bind them with `sources.py` to `FULL-RESULT.json` and `selected-result.json`.
3. Correct the axiom wording.
4. Restate or relabel the fusion lemma.
5. Drop the CP-SAT paragraph and the undefined totals.

If you want the big-integer brackets in Lean, bind `H` and `2228` to `certificates/frontier7000/CLAIM.json` and the admission receipt. Then compare bit lengths with `Nat.log2` or a `Nat.ble` evaluation that the kernel can reduce, not with `decide` on the full powers. Register the module so CI fails if it stops compiling. That leaves a small, accurate PR that CI can check.
