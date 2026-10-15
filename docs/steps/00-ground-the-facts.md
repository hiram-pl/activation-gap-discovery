# Step 0 — Ground the facts

**Owner:** both. Split as marked.
**Blocks:** Step 1, and therefore everything.
**Why this is a step and not a chore:** the plan currently rests on numbers
nobody on this team has read out of a paper. Two of them (the SAE expansion ratio,
the motivating case study) would change what we build or what we claim. The rest
are cheap to confirm while we are in the papers anyway. Doing this first costs a
day and prevents building on sand.

Nothing here needs a GPU. Both tracks can run in parallel.

---

## Track A — Paper facts (Person 1) · **DONE 2026-10-01**

Findings: [`docs/step0-trackA-findings.md`](../step0-trackA-findings.md), applied
to `PRD.md` in commit `3d2e394`.

Papers read: Dr. VLA 2603.19183v2, Fail2Drive 2604.08535v1, SimLingo 2503.09594v1,
RoboART 2502.06575v1, SAFE 2506.09937v2, RESample 2510.17640v4.

**What changed in the plan:**

1. **D1 closed — expansion ratio is 1, the brief was right.** With the paper's own
   reason quoted. The handoff was wrong. `PRD.md` §12.2.
2. **Dr. VLA code is released**, no LICENSE file. Step 7 becomes "configure an SAE
   stack" rather than "build one", but the causal check is still ours.
3. **The memorization filter is demoted to an ablation** — it would plausibly
   delete planted-gap features, since a gap looks memorized by its definition.
   `PRD.md` §9.2, `CONFLICTS.md` C17.
4. **B7's gap signal is our construction, not Dr. VLA's**, so it needs a written
   definition before Step 6 — and it may be stronger than we want. C18.
5. **SimLingo does not freeze the vision encoder**; our freezing it is a
   deviation and is now labeled as one. C19.
6. **RoboART's VLM critic is real** — the brief was right, this log's A5 had
   over-corrected. Restored.
7. **The motivating case study is confirmed verbatim**, including the mechanism,
   *but* a framing caution was added: `PedestriansOnRoad` is listable, so do not
   present it as an unlistable conjunction. `PRD.md` §3.2.

**Still open from Track A:**

- [x] **A5 · Event-grounded SAEs (2605.17204).** Read 2026-10-06 from the arXiv
      HTML; findings and four design consequences in `docs/step0-trackA-findings.md`.
      Spot-check the quotes against the PDF before citing them.
- [ ] Email the Dr. VLA authors about licensing (`PRD.md` §16.10).
- [ ] Items needing a loaded model or repo rather than a paper, now folded into
      Steps 1–2 (`PRD.md` §16.11). Closed: Qwen2-0.5B layer count and hidden size
      (2026-10-02), the Bench2Drive version (0.0.3), bucket weights, route-directory
      withholding and the Fail2Drive toolbox API (2026-10-06). Still open:
      PDMLite-F2D on a route XML we authored.

## Track B — Savio access (Person 2 primary, Person 1 confirms)

`PRD.md` §17. Person 1's account is already registered under `ic_cdss170fall`.

> **2026-10-05: GPU access exists.** The 2026-10-01 "no usable GPU partition"
> result came from asking for the `savio_normal` QoS, which the GPU partitions do
> not offer to `ic_` accounts. `sacctmgr show assoc` lists the allowance on
> `savio3_gpu` and `savio4_gpu` (`docs/WHAT_BROKE.md`, `configs/cluster/savio.yaml`).

- [x] **Person 2: complete the per-user steps.** Sign the cluster access agreement on
      first login; join the project under "My BRC Cluster Projects" in MyBRC; set
      up one-time-password auth. Login is a PIN plus a 6-digit code.
- [x] Both: SSH to `hpc.brc.berkeley.edu` successfully.
- [ ] Both: submit one trivial GPU job — `nvidia-smi` in a batch script is enough
      — and get output back. This is the actual done-condition. Everything else in
      Step 1 assumes it. **One of us has** (`scripts/check_gpu.sbatch`,
      2026-10-07, then Steps 1 and 2); the second is still open.
- [x] Record what GPU you landed on and under which partition: RTX A5000 on
      `savio4_gpu` for the GPU check, A40 on `savio3_gpu` for Steps 1 and 2.

### B2 · The four unknowns · do not infer

Do not infer these from general Research IT docs — allowance terms are set per
agreement, and we inherited cost estimates sized for different hardware.

1. ~~GPU partition names available to `ic_cdss170fall`.~~ The partitions above
   (`sacctmgr show assoc`).
2. The Service Unit charging rate for GPU jobs on those partitions. Step 1 and 2
   job charges are recorded in `PRD.md` §17.3; the rate itself is unconfirmed.
3. ~~Maximum wall time.~~ 8 hours (`sacctmgr show qos`, 2026-10-07).
4. Storage quota, and where large datasets should live. We need ~150 GB for
   activations at three layers, more if Mode A ever runs. Unknown.

Also shaping Step 4's fan-out: `savio_lowprio` is not associated with the
allowance; whether job arrays have a width cap is unknown.

### B3 · Environment reconnaissance

- [x] Does a CARLA 0.9.15 container exist on Savio, or do we build one? Check
      whether Singularity/Apptainer is the required runtime — most HPC sites
      disallow Docker. **Neither:** CARLA runs from the official Linux tarball
      (`CONFLICTS.md` C20, pending review).
- [x] Can CARLA render off-screen on a Savio GPU node? The handoff flagged this as
      a live risk on NRP and it is the most likely Step 1 blocker. **Yes**, Step 1
      on an A40, 2026-10-08.
- [ ] Where do the SimLingo checkpoint and dataset go, given the quota answer?
      The checkpoint lives under the clone's `.runtime/` on scratch
      (`configs/cluster/savio.yaml`); the quota answer is still missing.
- [ ] Note the module system's CUDA and Python versions.
- [ ] **Ray on Slurm.** Can a Ray cluster come up inside an allocation — head
      node plus one worker, `ray.init()`, one remote task returning a value? This
      is the smallest check that de-risks §11's orchestration choice. If it needs
      more than a day of fighting, we fall back to a plain job array; knowing that
      now is worth more than solving it now.

---

## Done condition

1. ~~Every unverified flag in `PRD.md` is confirmed or struck.~~ **Done** for the
   paper-answerable ones; the model/repo-answerable ones moved to §16.11.
2. ~~`CONFLICTS.md` D1 closed.~~ **Done — expansion ratio 1.**
3. [ ] **Both of us have run a GPU job on Savio and know what hardware we got.**
   One of us has; it no longer blocks Step 1, which passed on 2026-10-08.
4. ~~`docs/WHAT_BROKE.md` exists with its first entry.~~ **Done.**

**Status: Track A done; Track B mostly done (GPU access since 2026-10-05).
Open: Person 1's GPU job, and the SU rate and storage quota.**

## What we are explicitly not doing yet

No hooks, no SAE code, no gap predicates, no adapter interface. Step 1 is *one
route*, not the benchmark, and that is deliberate — the policy runs 15–20× slower
than real time and bring-up is the most likely place this project dies.

## Notes for whoever runs this

- If the Dr. VLA code did land on 1 Oct, say so loudly. It changes Step 7's scope
  from "build an SAE stack" to "configure one."
- If CARLA cannot render off-screen on Savio, stop and raise it. That is a
  replan, not a debugging session, and NRP becomes a serious option for rollouts
  rather than just the describability model.
- Log anything that surprises you in `docs/WHAT_BROKE.md` as you go. Reconstructing
  it later never works, and `PRD.md` §6 commits us to publishing it.
