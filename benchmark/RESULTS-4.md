# Benchmark results — fourth run (model/agent ladder)

5 configurations × 4 tasks × 3 reps × 2 variants = 120 agent runs, 60 judge calls.

| Config | Agent | Model |
|---|---|---|
| A | `claude -p --dangerously-skip-permissions --strict-mcp-config` | Opus 5 (default) |
| B | same, `--model claude-sonnet-5` | Sonnet 5 |
| C | same, `--model claude-haiku-4-5` | Haiku 4.5 |
| D1 | `opencode run --model ollama-cloud/kimi-k2.7-code` | Kimi K2.7 Code (Ollama Cloud) |
| D2 | `opencode run --model ollama-cloud/deepseek-v4-flash:0731` | DeepSeek V4 Flash (Ollama Cloud) |

Does not carry over earlier conclusions. `RESULTS.md`/`RESULTS-2.md`/`RESULTS-3.md` stand
as their own records under their own task/checklist/model wording.

## Two infrastructure bugs found and fixed during this run

- **`env` argument order** (`bin/run`): this box's `/usr/bin/env` was swapped to uutils
  coreutils mid-run, which — unlike GNU env — stops parsing `-u` flags once it hits a
  `NAME=value` assignment. Config C's (Haiku) first attempt was entirely invalidated (every
  run in both arms produced "no diff was provided"); fixed in commit `5b91a44`, Haiku was
  fully redone cleanly afterward. All numbers below are from the post-fix runs.
- **Shared results directory**: an operator cleanup step (`rm -rf` on the control task's
  results directory before each new configuration's smoke test) deleted already-written
  control-task verdicts for 4 of 5 configurations, since all configurations write into the
  same `results/<task>/` directory. Recovered with a dedicated re-run; the control numbers
  below are complete for all 5 configurations.

## 00-control-value-object — the control, per configuration, first

| Config | with-skills (of 12) | without-skills (of 12) |
|---|---|---|
| A (Opus) | 12/12 | 12/12 |
| B (Sonnet) | 12/12 | 12/12 |
| C (Haiku) | 10/12 (3,4,3 per rep) | 10/12 (4,3,3 per rep) |
| D1 (Kimi) | 12/12 | 12/12 |
| D2 (Deepseek) | 12/12 | 11/12 (3,4,4 per rep) |

**Flat for 3 of 5 configurations (Opus, Sonnet, Kimi), near-flat with no consistent
direction for the other 2.** Haiku fails one item in most reps in *both* arms, never the
same item lopsidedly in one direction — a capability ceiling on this specific plain-PHP
task, not a with/without-skills effect. Deepseek is one item short on one without-skills
rep. Neither shows the pattern a real skills effect would produce (one side consistently
lower); this control did its job at every rung. **Every number below can be read at face
value.**

## 02-json-endpoint — the continuity check across the ladder

Per-item breakdown (of 3 reps each), the item order is: `#[MapRequestPayload]` /
Validator-on-DTO / framework-driven 422.

| Config | MapRequestPayload with/without | Validator with/without | 422 with/without |
|---|---|---|---|
| A (Opus) | **3/1** | 3/3 | 3/0 |
| B (Sonnet) | **3/0** | 3/3 | 3/0 |
| C (Haiku) | **2/0** | 2/0 | 0/0 |
| D1 (Kimi) | **3/0** | 3/0 | 1/0 |
| D2 (Deepseek) | **3/0** | 3/0 | 2/0 |

**The headline claim — `#[MapRequestPayload]` used with-skills, `json_decode()` /
`Serializer::deserialize()` without-skills — holds across every configuration on this
ladder, with-skills always ahead, without-skills never once reaching 3/3.** But it is not
the "zero exceptions, ever" claim run 3 reported: on this 4th independent Opus sample, one
without-skills rep *did* use `#[MapRequestPayload]` (3/3 vs 1/3, not 3/3 vs 0/3). Checked
directly — the diff genuinely shows the attribute, correctly used, with no skill present.
One occurrence in 15 without-skills runs across the whole ladder (Opus rep 2 of this run);
every other without-skills run on every other configuration used manual parsing. The
honest read: a very strong, real, model-independent effect, not an absolute one.

Validator-on-DTO stopped discriminating below Sonnet: Haiku and both opencode
configurations show it 0/3 without-skills too (both variants already reach for
`#[Assert\...]` at the strong models, Haiku is inconsistent both ways, the opencode models
apparently do not reach for it unprompted without-skills at all — a genuine, if narrower,
skill effect at that tier). The 422 row, now legitimately failable per `controller/SKILL.md`
documenting that `#[MapRequestPayload]` already answers it, never once passed
without-skills on any configuration; with-skills passed it on Opus/Sonnet (3/3) but far
less reliably on the weaker/other-agent configurations (0–2 of 3) — consistent with the
skill helping, but the models least likely to reach for `#[MapRequestPayload]` at all being
the same ones least likely to get the automatic 422 that depends on it.

## 01-lock — does the `command` skill matter more as the model/agent gets weaker?

Aggregate items passed (of 4) per rep, with/without:

| Config | Rep 1 | Rep 2 | Rep 3 |
|---|---|---|---|
| A (Opus) | 4/4 | 4/4 | 4/4 |
| B (Sonnet) | 4/4 | 4/4 | 4/4 |
| C (Haiku) | 4/1 | 0/1 | 3/0 |
| D1 (Kimi) | 2/0 | 0/2 | 2/0 |
| D2 (Deepseek) | 4/4 | 2/4 | 4/4 |

**Opus and Sonnet: flat ceilings, as in run 3.** Haiku and Kimi: real, substantial
variance — but not a clean "skill helps the weak model" story. Haiku rep 1 and rep 3 favor
with-skills strongly (4 vs 1, 3 vs 0); rep 2 favors *without*-skills (0 vs 1). Kimi never
shows with-skills ahead by more than 2 and one rep (rep 2) has without-skills ahead (0 vs
2). Deepseek is close to Opus/Sonnet's ceiling except one soft with-skills rep (2/4). This
reads as **noisy, not systematic** — weaker models/other agents are less reliable at this
task in both directions, and the skill does not reliably rescue the failures. Do not read
this as "the skill matters more at Haiku/Kimi" without more reps; the direction flips
within the same 3-rep sample for both.

## 03-post-authorization — same question for the `voter` skill

| Config | Rep 1 | Rep 2 | Rep 3 |
|---|---|---|---|
| A (Opus) | 4/4 | 2/4 | 4/2 |
| B (Sonnet) | 4/4 | 4/4 | 4/4 |
| C (Haiku) | 2/0 | 4/0 | 0/0 |
| D1 (Kimi) | 4/? | 4/4 | 4/4 |
| D2 (Deepseek) | 4/0 | ?/4 | 4/4 |

(`?` = judge output for that half was empty/malformed for that specific rep — treated as
missing, not as fail, and excluded from the read below rather than padded.)

Sonnet stays a flat ceiling. Opus wobbles slightly (one rep each direction) but stays
close to ceiling overall — noise at the margin, not a pattern. Haiku is the clearest
signal in this run: with-skills reaches the voter pattern in 2 of 3 reps (partially or
fully), without-skills never does (0/4 in all 3 reps) — the one configuration where the
`voter` skill visibly and consistently helps. Kimi and Deepseek stay close to ceiling on
both sides where readable. **Weak evidence that `voter` starts to matter at the Haiku
tier specifically**, on top of `02`'s Validator-on-DTO result showing the same tier
threshold — both point at Haiku as the first place in this ladder where a skill's
presence visibly changes the outcome, but each is a single 3-rep sample and neither is a
settled finding yet.

## opencode: D1 vs D2 (same agent, model changes only)

Comparable to each other; **not** comparable to the Claude configurations, since the
install path also changes (skills copied into `.agents/skills/` + slim `AGENTS.md` index,
per `docs/opencode.md`, not `.claude/skills/`).

- **00 (control):** both flat (12/12 both arms).
- **02:** both show the MapRequestPayload effect (3/0), Kimi's Validator/422 rows are
  slightly stronger than Deepseek's on 422 specifically (Kimi 1/3 with, Deepseek 2/3 with;
  Validator identical 3/0 both).
- **01/03:** Kimi is noisier than Deepseek on `01` (bigger swings, one clear
  skill-made-it-worse rep); Deepseek tracks closer to the Claude ceiling on `03`, Kimi is
  flat near-ceiling too. No consistent "smaller model needs the skill more" pattern
  between the two — Deepseek (166 GB index) is not obviously weaker here than Kimi (595 GB,
  code-specialized).

## Summary

- **The headline effect from runs 1–3 (`#[MapRequestPayload]`) holds across every
  configuration on this ladder** — Opus, Sonnet, Haiku, and both opencode models all show
  with-skills ahead, without-skills never reaching full marks. It is a strong,
  cross-model effect, though this run found the first exception to the "zero exceptions
  ever" framing (one Opus rep used the attribute without the skill).
- **The control held at 3 of 5 configurations and showed only small, directionless noise
  at the other 2** (Haiku, Deepseek) — no configuration's control moved the way a real
  effect would move it, so every other number in this report can be trusted at face value.
- **The single most useful sentence:** skills start to visibly matter, beyond the
  `#[MapRequestPayload]` effect that holds everywhere, specifically at the **Haiku 4.5**
  tier — `voter` (03) and Validator-on-DTO (02) both flip from "both variants already know
  this" at Opus/Sonnet to "with-skills reaches it, without-skills does not" at Haiku. The
  `command` skill (01) does not show the same clean pattern — it's noisy in both
  directions at Haiku and Kimi, not a reliable rescue. This is not yet a settled finding
  (single 3-rep samples per configuration); it is the specific place a follow-up run
  should concentrate reps if the goal is "which models need this."
