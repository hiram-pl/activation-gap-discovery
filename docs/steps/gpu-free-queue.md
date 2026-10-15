# GPU-free work queue

Work that needs no GPU. Everything here runs on a laptop: no GPU, no cluster, no
partition name, no rollouts.

**Where it came from.** This queue was written on 2026-10-01, when Savio GPU
access looked blocked. It was not: the jobs had asked for the wrong QoS, and the
allowance has had GPU partitions all along (corrected 2026-10-05,
`docs/WHAT_BROKE.md`; Steps 1 and 2 ran on Savio on 2026-10-08). The items stay
useful because they need no GPU, not because the GPU is unavailable.

**Why this is not filler.** Most of it is work that has to happen anyway, and
some of it is better done early. The scoring layer in particular wants to be
frozen and hash-committed *before* the test pool exists (`PRD.md` §9.1), so
building it first was the correct order, not a consolation prize.

**Standing constraint:** nothing here may assume a GPU model or a partition
name. Cluster facts live in `configs/cluster/savio.yaml` and are read, never
hardcoded.

---

## Done

### ✅ Scoring and metrics · `src/harness/scoring.py`
The headline metric, testable on synthetic data. Hungarian assignment over all
gaps at once, per-tier recall, precision@M dividing by the budget, mean best
Jaccard, decoy hit rate, validated precision, and the §13.1 harness-validity
check. 32 tests in `tests/`.

Three behaviours worth knowing, because each is a trap avoided:
- `jaccard(∅, ∅) == 0`, not 1 — otherwise a method scores perfectly by emitting
  nothing.
- One Hungarian assignment, not greedy per-gap best match — otherwise one broad
  axis gets credited for two gaps.
- Precision divides by M, never by the number of axes emitted — otherwise
  emitting a single safe axis beats a full directive.

### ✅ Data contracts · `src/harness/schema.py`
Axis card, directive, gap. A directive that does not validate cannot enter
scoring, so a malformed baseline fails loudly instead of scoring zero and
looking like a real negative result. Empty `name` or `description` is a hard
error: an axis without them is a cluster, not a discovery (`PRD.md` §8.3).

### ✅ Sealed-key CI guard · `scripts/check_sealed_imports.py`
The mechanical half of what blinding used to buy socially. Catches both import
and string-path leaks under `discovery/`, `baselines/`, `gate/`. Tested by
injecting real leaks. Docstrings mentioning the rule are exempt — false
positives train us to ignore the checker.

### ✅ Config plumbing · `src/harness/config.py`, `scripts/check_configs.py`
Content hashing (`PRD.md` §6.2) and `TBD-` placeholder detection, so a run
cannot silently proceed on a placeholder.

### ✅ Cluster indirection · `configs/cluster/savio.yaml`, `scripts/submit.sh`
Single place any partition or GPU type is named. `submit.sh` refuses GPU
submission while `gpu.available: false` and prints why. Both GPU profiles
(`gpu`, `gpu_a40`) are `available: true` since 2026-10-05.

### ✅ B1 random baseline · `src/baselines/random_baseline.py`
The only baseline that needs no policy at all: draw M random subsets of failing
rollout ids, sized from the primary method's axis sizes. It exercises the whole
scoring path with a real method object rather than a test fixture, and emits a
valid `Directive`. Closes a line item from `PRD.md` §9.3.

---

## Next, in value order

### 1 · B7's gap-signal definition · `PRD.md` §16.9, `CONFLICTS.md` C18
**Drafted 2026-10-09, awaiting approval:** `docs/steps/b7-gap-signal.md`.

**Must exist before Step 6 or B7 is not a fair baseline.** Dr. VLA never proposes
memorized-feature concentration as a gap signal — that construction is ours, so
we owe it a written definition: how concentration is measured, over what
partition, how it ranks into axes.

Writing it before anyone has seen the activations is also the honest order: we
cannot tune it to win or lose after the fact. Activations have been recorded on
the Mac and on Savio since 2026-10-06 and stay unopened until the draft is
approved.

### 2 · Synthetic end-to-end dry run
Generate a fake rollout corpus with known planted structure, run B1 and a stub
discovery method through scoring, and produce the actual report artifact. This
smokes out the whole pipeline's plumbing and gives us the results-table format
before real data exists.

It also answers a question we should not discover late: what does the scoring
report look like when a method recovers nothing? The negative-result paper needs
that table to be legible.

### 3 · `harness/PREREGISTRATION.md` skeleton
The §13.1 go/no-go rule, the efficacy thresholds, the gate AUROC threshold, the
primary-method declaration. **The entire value of pre-registration is that it is
written before the numbers exist**, so it must be drafted before Step 4's sweep
produces any. Fill the thresholds now; leave only the primary-method choice
open, since that legitimately depends on the Step 4 sweep.

### 4 · Gap predicate schema and the public practice gaps
The predicate DSL from `PRD.md` §9.1, plus the two public practice gaps that live
in the dev pool. Needs no rollouts — only a schema and a matcher that can be
tested against synthetic metadata.

Keep sealed predicates out of this; only the practice gaps are public.

### 5 · Describability gate scaffolding
The two-model protocol with its shuffle control (`PRD.md` §9.2). The prompt
templates, the held-out split logic, and the AUROC computation are all
testable on synthetic descriptions. The actual VLM calls need NRP, not Savio.

### 6 · Results database schema
Config hash → run record → metrics. `PRD.md` §6.2 requires every paper number to
be regenerable from a query. Pure schema work.

---

## Deliberately not in this queue

- **The adapter interface.** `PRD.md` §8.2 says explicitly to build it by
  refactoring after SimLingo works. Designing it now would be the abstraction
  mistake the brief warned about. That refactor is Step 11.

Three items listed here on 2026-10-01 have since moved outside the queue:

- **The SimLingo checkpoint.** Strict-loaded on CPU and MPS on 2026-10-02, then
  on a Savio A40 in Step 1; the architecture facts (`PRD.md` §16.11) are closed.
- **Ray orchestration.** Not used so far: Step 2's 220 routes ran as a plain
  Slurm job array (`scripts/route_sample.sbatch`), two routes per A40. Whether
  later steps still need Ray (`PRD.md` §11) is not decided.
- **Container work.** Replaced: Savio runs CARLA from the release tarball
  (`CONFLICTS.md` C20, pending review).
