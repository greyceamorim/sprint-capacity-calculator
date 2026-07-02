# Sprint Capacity Calculator

A lightweight, single-file tool for forecasting a team's realistic delivery capacity for a sprint — in **dev-days**, **hours**, **effective FTE**, and optionally **story points**. It runs entirely in the browser, with no build step and no dependencies beyond a web font.

Built as a personal project. The interface is generic and ships with example data only.

---

## Why this exists

Most capacity planning silently treats **presence as productivity** — if a person is "on the team", they're counted as a full unit of output. That works for a stable, fully-ramped squad and quietly overstates capacity in the everyday cases that matter most:

- **Part-time or shared members** split across more than one team.
- **New joiners on the learning curve**, who are present but not yet delivering at full output.
- **Time lost to non-delivery work** — reviewing, mentoring, coordination, support — that doesn't turn into shipped work.

This tool separates two things people usually blur together:

- **Capacity** — time available. A new joiner has the same ten days as anyone else.
- **Productivity** — how much of that time turns into finished work, which varies with experience, ramp-up, and how much of someone's time goes to supporting others rather than delivering.

The calculator measures capacity precisely and then gives you one honest lever — **allocation %** — to express effective contribution rather than mere attendance. It never pretends to know a team's productivity; it makes the assumptions visible so a planning conversation can happen around them.

---

## Features

- **Per-person breakdown.** Each member is a bar showing exactly how raw days erode down to net delivery: public holidays and leave, part-time allocation, then the focus factor.
- **Multi-team tabs.** Add as many teams as you like, each with its own roster, sprint settings, and colour. Rename inline; recolour from a preset palette.
- **Two planning units, side by side.** Net capacity in **dev-days** and, for teams that plan in points, **story point capacity** as a headline number. The point metric greys out for teams that plan in dev-days.
- **Capacity gauge.** Shows net delivery as a share of the team's theoretical maximum, so erosion is visible at a glance.
- **Over-allocation warning.** Flags allocation over 100% or leave longer than the sprint.
- **Auto-save.** State persists between sessions on the same device (see *Data & storage* for the current limitation).

---

## Getting started

Open the `.html` file in any modern browser. That's it — there's nothing to install or build.

- `sprint-capacity-calculator-v2.html` — current version (multi-team, dev-days, story-point headline).
- `sprint-capacity-calculator.html` — original single-team version, kept for reference.

---

## How the maths works

For each team member, per sprint:

```
theoretical_max = working_days − public_holidays        (per person)
after_leave     = theoretical_max − personal_leave
after_alloc     = after_leave × (allocation% / 100)
net_dev_days    = after_alloc × (focus_factor% / 100)
```

The four bar segments always sum to `theoretical_max`:

| Segment | Meaning |
| --- | --- |
| Net delivery | `net_dev_days` — real work on the sprint backlog |
| Lost to focus factor | ceremonies, context-switching, BAU / support |
| Lost to allocation | time the person spends outside this team |
| Leave & holidays | personal leave plus public holidays |

Team totals:

```
net_total   = Σ net_dev_days
total_hours = net_total × hours_per_day
effective_FTE = net_total ÷ (working_days − public_holidays)
utilisation   = net_total ÷ Σ theoretical_max
story_points  = round(net_total × points_per_dev_day)   (optional)
```

---

## Story points vs dev-days

The tool serves both planning styles from the same numbers:

- **Teams that plan in dev-days** (e.g. data science) read **net capacity** directly and leave *Pts / dev-day* blank.
- **Teams that plan in story points** enter their historical **Pts / dev-day** ratio, and the tool converts capacity into a suggested point target.

### Deriving `Pts / dev-day` from historical velocity

The ratio comes from your own delivery history:

```
points_per_dev_day = average_velocity ÷ average_net_dev_days
```

Example: a team averaging **30** points over recent sprints, delivered by **3 developers** at 80% focus over a 10-day sprint, has `3 × 10 × 0.80 = 24` net dev-days, so `30 ÷ 24 ≈ 1.25` points per dev-day.

Because velocity is the *actual* output the team produced, it already absorbs leave, holidays, ceremonies, and interruptions. You don't re-adjust the average for those; you let the calculator flex each individual sprint around that baseline.

### `Pts / dev-day` and focus factor are the same reality

They are two dials on the same underlying truth. If you trust the focus factor you set, history tells you the resulting pts/dev-day. Fix the ratio instead and the same arithmetic returns the focus factor. Historical velocity lets you solve for one using the other — a calibrated focus factor from data rather than a guess.

### Handling a changing team

The derived ratio is an **average over the team mix you had**. When the mix changes — someone new joins, or an experienced member starts spending time supporting others — that average no longer applies to the new composition, but the tool still multiplies it across everyone. To stay honest, lower the *allocation %* to reflect effective contribution, and account for the cost on both sides:

- A member still ramping up: a fraction (e.g. 0.50), rising over subsequent sprints as they reach full output.
- Anyone whose time shifts into mentoring, review, or support: below 100% (e.g. 0.75) while that lasts.
- Fully-ramped members delivering as usual: 1.0.

This keeps a newly-grown team from inflating its forecast before it actually delivers the extra output.

---

## Data & storage

All data stays in the browser — nothing is sent anywhere. There is no backend and no analytics.

**Current limitation:** persistence uses a storage API specific to the environment the tool was authored in. In a standard browser (including when served from static hosting such as GitHub Pages) that API is absent, so the tool runs but does **not** persist between sessions. Swapping the storage layer to `localStorage` is the first task before hosting — see the roadmap. The storage calls are isolated in `load()` / `save()`, so the change is contained.
