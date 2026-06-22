# PLANETARY DEFENCE OFFICER
## Agent Working Document — Project Plan & Phased Execution

```
╔══════════════════════════════════════════════════════════════════╗
║  P L A N E T A R Y   D E F E N C E   O F F I C E R              ║
║  ESA Near-Earth Object Coordination Centre — NEOCC Simulator    ║
║  ─────────────────────────────────────────────────────────────  ║
║  AGENT WORKING DOCUMENT — v1.0                                  ║
║  Successor project to STELLaRUM (SDSS DR17 Classification)      ║
╚══════════════════════════════════════════════════════════════════╝
```

---

# SECTION 0 — How to use this document

You are an AI coding agent working alongside the project author (referred to
hereafter as *the operator*) on a two-part academic portfolio project: a
machine-learning pipeline (backend) and a browser-based simulation game (frontend)
themed around ESA's Near-Earth Object Coordination Centre.

This document is structured in three layers:

- **Sections 1–7** are reference material. They describe what the project is,
  how it is structured, what the visual language is, what data flows where,
  and what the gameplay loop looks like. Consult them when you need context.
- **Section 8** is the **phased execution plan**. This is the working part of
  the document. You proceed one phase at a time, in order. Each phase has a
  clear goal, deliverables, exit criterion, and an explicit "Do NOT" list of
  things that belong to later phases.
- **Sections 9–11** are operating constraints — scope split, working rules,
  open questions.

### Critical operating rules

1. **One phase at a time.** Complete a phase fully (its exit criterion is met)
   before moving to the next. Do not bleed work from later phases into earlier
   ones, even if it seems easy or convenient.
2. **Pause for review at phase boundaries.** When a phase is complete, summarise
   what was done, what the exit criterion shows, and wait for the operator to
   confirm before starting the next phase.
3. **Reference sections are reference.** Do not implement everything they describe
   immediately. They describe the *final state*, not the next step.
4. **When in doubt, ask the operator.** Especially on design decisions, naming,
   game balance numbers, or anything in Section 11 (open questions).

---

# SECTION 1 — Project context

## What we are building

A bureaucratic-simulation game in the lineage of *Papers, Please*, set inside
ESA's Near-Earth Object Coordination Centre (NEOCC) in Frascati. The player
acts as a Planetary Defence Officer reviewing incoming reports on Near-Earth
Objects and issuing one of three verdicts per object: **SAFE / MONITOR /
DANGEROUS**. Reports arrive as fragments from multiple observatories
(Pan-STARRS, ATLAS, Catalina, JPL-CNEOS) interleaved in a single telemetry
feed. The player must collect the right fragments for the right object,
assemble them on a worksheet, and deliver a verdict before the object's
approach window closes.

The cognitive load — the source of all gameplay tension — comes from
**Variant (a) error mechanic**: fragments in the telemetry feed are tagged
with the object they belong to, but there are many fragments, they arrive
continuously, numerical values across objects are often similar, and it is
easy to copy a fragment from *the wrong object* into the active worksheet.

## Why this project

The project is the successor to **STELLaRUM**, a completed SDSS DR17 stellar
classification project with a radio-telescope frontend game (single-file HTML,
~2600 lines, vanilla JS, custom k-means, CRT/VHS aesthetic). It establishes a
deliberate portfolio series: each entry pairs a real astronomical dataset with
a thematically appropriate browser-based interaction, with consistent visual
language across the series.

Portfolio relevance: ESA NEOCC is a real facility; ESA's Hera mission
(asteroid deflection follow-up to NASA's DART) launched October 2024;
planetary defence is an active ESA priority area.

## Backend / frontend split

```
backend   →  trains classifiers on NASA NEO data, generates synthetic
             borderline objects (the "Evil Model"), exports a static
             JSON datapackage of ~3000 game-ready objects with model
             predictions, confidences, and source attributions

frontend  →  single HTML file that loads the JSON datapackage at startup,
             runs the telemetry-feed game loop entirely in the browser,
             no server, no backend HTTP calls at runtime
```

## Dataset

**Primary:** `lovishbansal123/nasa-asteroids-classification` (or equivalent
`shrutimehta` version) on Kaggle. ~4,687 objects, 40 features, sourced from
NASA's NEOWS (Near-Earth Object Web Service). Contains:

- Orbital elements: semi-major axis (`a`), eccentricity (`e`), inclination (`i`),
  perihelion distance (`q`), aphelion distance (`Q`)
- Proximity data: MOID (Minimum Orbit Intersection Distance), close approach
  date, relative velocity
- Physical characterisation: absolute magnitude (`H`), estimated diameter
  (min/max)
- Risk scales: Palermo Scale, Torino Scale
- Target: `Hazardous` (boolean → 0/1)

The class imbalance in this dataset is severe (most objects non-hazardous) —
this is a feature of the project, not a bug. The whole `class imbalance →
SMOTE → game asset generation` narrative depends on it.

---

# SECTION 2 — Visual identity

The visual language is **inherited directly from STELLaRUM**. This is
deliberate — the projects form a portfolio series, and consistency across them
is a signal of intentional design, not laziness. Where STELLaRUM established
something that works, copy it verbatim and do not re-invent.

## Palette

```
--bg        : #000000   /* near-black background                */
--surface   : #080808   /* panels, dossier backgrounds          */
--border    : #1a1a1a   /* dividers, faint frames               */
--border-2  : #363636   /* prominent frames, panel edges        */
--dim       : #444444   /* tertiary text, decorative            */
--muted     : #707070   /* secondary text, labels               */
--text      : #c0c0c0   /* primary readout text                 */
--bright    : #dedede   /* active element text                  */
--white     : #f0f0f0   /* logo, header, top-level emphasis     */

/* SEMANTIC ACCENTS — new for planetary defence */
--ok        : #6aac79   /* SAFE verdict, copied from STELLaRUM  */
--warn      : #c8a235   /* MONITOR verdict                      */
--crit      : #c84040   /* DANGEROUS verdict, PHA confirmed     */
--redacted  : #2a2d35   /* blacked-out field background         */
```

## Typography

```
--font-mono : 'Share Tech Mono', 'Courier New', monospace   /* body */
--font-vt   : 'VT323', monospace                             /* logos, splash */

Base size: 13px, line-height 1.4
```

No decorative fonts. No gradients. No glow effects except where STELLaRUM
already established them (logo text-shadow on splash).

## CRT / VHS overlay

Copy from STELLaRUM directly, verbatim:

- SVG `feColorMatrix` chromatic aberration filter applied to `#app` (red
  channel offset −1px, blue channel offset +1px)
- Horizontal scanline overlay via `repeating-linear-gradient`
- RGB vertical strings overlay with `mix-blend-mode: screen` and a
  `vhsShift` animation
- White scan-line bar travelling top-to-bottom every 7s (`vhsScan` animation)
- Radial vignette via `body::after` pseudo-element

These exist as a single CSS block in STELLaRUM (`/* ============ VHS / CRT
— AGGRESSIVE ============ */`). Lift it as-is.

## Iconography

Reuse glyph vocabulary from STELLaRUM where applicable:
- `◈` for section headers in panel titles
- `✦` for STAR-type objects (not used here but reserved for series consistency)
- `░` `▒` `▓` for fill bars and ASCII flourishes
- Box-drawing characters `═ ║ ╔ ╗ ╚ ╝` for splash screen ASCII art

For planetary defence specifically:
- `⊙` Sun, used in orbital diagrams (v2)
- `◉` PHA / hazardous indicator
- `●` regular object indicator
- `░░░░` REDACTED field placeholder

---

# SECTION 3 — Project architecture

```
planetary-defence/
│
├── README.md                         # Public README, structured like
│                                       STELLaRUM's README. Final polish
│                                       at the very end of the project.
│
├── AGENT.md                          # This document.
│
├── backend/
│   ├── 01_eda.ipynb
│   ├── 02_preprocessing.ipynb
│   ├── 03_baseline_models.ipynb
│   ├── 04_imbalance.ipynb
│   ├── 05_moid_ablation.ipynb
│   ├── 06_evil_model.ipynb
│   ├── 07_export.ipynb
│   ├── requirements.txt
│   └── output/
│       ├── dataset_full.npz          # Scaled features + labels
│       ├── dataset_noMOID.npz        # Ablation feature set
│       ├── rf_full.pkl               # Trained RF, full features
│       ├── rf_noMOID.pkl             # Trained RF, no MOID
│       ├── metrics.json              # All model metrics, all experiments
│       ├── feature_importances.json
│       ├── synthetic_pool.json       # Raw SMOTE output before validation
│       ├── game_objects.json         # FINAL game datapackage (~3000 obj)
│       └── *.png                     # Plots: EDA, confusion, importance,
│                                       SMOTE before/after, etc.
│
└── frontend/
    ├── planetary_defence.html        # Single-file entry point
    └── assets/
        └── data/
            └── game_objects.json     # Copied from backend/output/
```

**Single-file frontend** — same as STELLaRUM. All HTML, CSS, JS in one file.
No build step. No npm. No bundler. The only external dependency is the Google
Fonts CDN for `Share Tech Mono` and `VT323`.

---

# SECTION 4 — Data contract (backend → frontend)

This is the JSON schema produced by `07_export.ipynb` and consumed by
`planetary_defence.html`. **Both sides of the project must agree on this
contract before any frontend gameplay logic is written.**

## game_objects.json structure

```json
{
  "meta": {
    "generated_at": "2026-MM-DD",
    "n_real_safe": 2000,
    "n_real_hazardous": 250,
    "n_synthetic_hazardous": 750,
    "model": "RandomForest_borderlineSMOTE",
    "model_accuracy": 0.978,
    "model_pr_auc_hazardous": 0.88,
    "seed": 42
  },
  "objects": [
    {
      "object_id": "2026-XB7",
      "is_synthetic": false,
      "true_class": "hazardous",
      "model_prediction": "hazardous",
      "model_confidence": 0.87,

      "fields": {
        "orbital": {
          "a": 1.42,
          "e": 0.61,
          "i": 4.2,
          "q": 0.55,
          "Q": 2.29
        },
        "proximity": {
          "moid": 0.043,
          "close_approach": "2031-04-12",
          "rel_v": 19.8
        },
        "physical": {
          "H": 22.4,
          "diameter_min": 0.35,
          "diameter_max": 0.78
        },
        "probability": {
          "palermo": -2.15,
          "torino": 1,
          "impact_prob": 0.00012
        }
      },

      "source_attribution": {
        "orbital":     "PAN-STARRS",
        "proximity":   "ATLAS",
        "physical":    "CATALINA",
        "probability": "JPL-CNEOS"
      }
    }
  ]
}
```

### Field group meaning

Each top-level group in `fields` maps to one **document type** in the game:

- `orbital` → DOC TYPE 1 (Orbital Survey Report)
- `proximity` → DOC TYPE 2 (Proximity Analysis)
- `physical` → DOC TYPE 3 (Physical Characterisation)
- `probability` → DOC TYPE 4 (Probability Update)

### Source attribution

`source_attribution` assigns one of {`PAN-STARRS`, `ATLAS`, `CATALINA`,
`JPL-CNEOS`} to each field group, **pseudo-randomly per object**. This is what
produces the telemetry-feed confusion: the player sees lines like
`[03:14:22] PAN-STARRS // 2026-XB7 // a=1.42 e=0.61 i=4.2` and must keep
track of which `object_id` each fragment belongs to.

The randomisation must be seeded (`seed: 42`) so the same export reproduces
the same source assignments — important for debugging.

### `object_id` format

Use realistic-looking NEO designations: `2026-XB7`, `2019-AQ3`, `1998-KY26`,
`2031-FG1`. These follow real Minor Planet Center provisional designation
syntax (year + letter pair + optional number). Backend generates these
deterministically from the dataset's actual `Neo Reference ID` field where
available, falling back to synthetic IDs for SMOTE-generated objects.

---

# SECTION 5 — ML pipeline reference

This section describes the *final state* of the seven notebooks. The
**phased execution plan in Section 8 maps each notebook to one or two
phases**. Do not implement multiple notebooks in one phase.

## Stack

```
python 3.11+
pandas, numpy
scikit-learn         # LogReg, KNN, RandomForest, metrics, preprocessing
xgboost
imbalanced-learn     # SMOTE, BorderlineSMOTE, ADASYN
scipy                # ConvexHull, Delaunay for physical validation
matplotlib, seaborn
jupyter
```

No TensorFlow / PyTorch. No AutoML. No hyperparameter optimisation
libraries beyond simple GridSearchCV if needed (and likely not needed).

## Models

| Model | Role | Notebook |
|---|---|---|
| LogisticRegression | Linearity baseline | 03 |
| KNN (k=5) | Geometric baseline | 03 |
| RandomForest | Primary production model | 03–07 |
| XGBoost | Performance ceiling reference | 03 only |

**Random Forest** is the model that:
- Powers all downstream experiments (04–07)
- Provides feature importances
- Generates predictions exported to the frontend

XGBoost is run once in notebook 03, the comparison is noted, and it does not
reappear later. This is deliberate — the project narrative is around
interpretability (feature importances, MOID ablation), and RF serves that
narrative more cleanly than gradient-boosted trees.

## Metrics — what to always report

For every model run, in every notebook:

- Overall accuracy
- **PR AUC for the hazardous class** (primary metric — accuracy alone is
  meaningless on imbalanced data)
- Per-class precision, recall, F1
- Confusion matrix as a plot

Never report only accuracy. Never use ROC AUC as the primary metric on this
imbalanced dataset (use it as a secondary if you want, but call out that PR
AUC is more honest for the imbalance regime).

## Class imbalance handling — the central narrative

```
Stage 1: Naive RF, no resampling
         → ~98% accuracy, ~20-30% hazardous recall
         → "the failure mode" — model learns to always predict SAFE

Stage 2: class_weight='balanced'
         → quick cheap fix, recall improves, accuracy drops
         → demonstrates the trade-off

Stage 3: Borderline-SMOTE
         → primary solution, generates synthetic minority on the decision
           boundary, F1 hazardous rises substantially

Stage 4: ADASYN
         → optional comparison if time permits, otherwise note it exists
```

Choice of resampling strategy is made in **notebook 04 based on metrics**,
not predetermined. Borderline-SMOTE is the strong default; if ADASYN wins
empirically, use ADASYN. Document the choice clearly.

## Synthetic object generation — physical validation

In **notebook 06**, after SMOTE generation, apply this four-step physical
validation pipeline:

### Step 1 — Split independent vs derived features before SMOTE

Apply SMOTE only on independent physical parameters:
```
a, e, i, MOID, H, diameter_min, rel_v, albedo
```

Compute derived parameters **after** SMOTE using physical formulas:
```
q             = a × (1 − e)             # perihelion
Q             = a × (1 + e)             # aphelion
diameter_max  = f(H, albedo)            # via brightness formula
```

If you SMOTE everything together, `q` and `Q` will drift inconsistent with
`a` and `e`. The resulting "synthetic asteroids" become mathematical
nonsense.

### Step 2 — Hard physical clamps after SMOTE

```
0 ≤ e < 0.99                  # avoid parabolic/hyperbolic — these are
                              # interstellar interlopers, not NEOs;
                              # real NEO dataset doesn't go above ~0.97
0 ≤ i ≤ 180                   # inclination is bounded
a > 0, diameter > 0, MOID ≥ 0 # physical positivity
rel_v ∈ [5, 40] km/s          # realistic NEO velocity range
H ∈ [10, 33]                  # astronomically meaningful magnitude
```

Synthetic objects violating any clamp are **dropped, not clipped** — clipping
introduces edge-pile-up artifacts.

### Step 3 — Convex hull validation

Use `scipy.spatial.ConvexHull` + `Delaunay.find_simplex` to verify each
synthetic point lies inside (or close to) the convex hull of real NEO
points in feature space. Drop points far outside the hull — these are
extrapolations beyond the training distribution.

### Step 4 — Visual sanity check (mandatory)

Generate a pair plot of real PHA vs synthetic borderline on the main
features. The synthetic points must lie *among* the real ones visually,
not in a separate cloud. Save plot as `output/synthetic_validation_pairs.png`.

If synthetic forms a visibly separate cluster: stop, debug, do not export.

### Expected attrition

Expect ~30-50% of raw SMOTE output to fail validation. This is normal and
itself a useful portfolio metric — it shows you understand the method's
limitations rather than trusting it blindly.

## Final pool composition

Backend ultimately produces ~3000 game-ready objects:

```
1000 hazardous = 250 real PHA + 750 synthetic borderline
2000 safe      = random sample from real non-hazardous
```

The hazardous pool is deliberately weighted toward synthetic. Real PHA in
the dataset tend to be unambiguous (large diameter, low MOID, classifier
confidence > 0.95) — they are not borderline. Borderline-SMOTE generates on
the decision boundary by definition, where confidence is 0.5–0.7. These are
the cognitively demanding objects for the player. If the dangerous pool is
dominated by easy real PHA, gameplay difficulty collapses.

The 1:2 hazardous:safe ratio is a deliberate gameplay choice — the real
dataset is ~9.7% hazardous, but ~10% danger rate is too low for a game where
players need regular threat encounters. 33% gives a realistic-feeling threat
rate without making every other object dangerous.

---

# SECTION 6 — Game design reference

## The two-screen metaphor

The interface emulates a NEOCC operator's twin-monitor workstation. The
camera slides horizontally between two logical screens:

```
┌─────────────────────────────────┐  ┌─────────────────────────────────┐
│                                 │  │                                 │
│   SCREEN 1 — MONITORING         │  │   SCREEN 2 — WORKSHEET          │
│                                 │  │                                 │
│   Telemetry feed (left col)     │  │   Active dossier (centre)       │
│   Dossier list (centre)         │  │   Field slots: orbital,         │
│   Approach timeline (bottom)    │  │   proximity, physical,          │
│   Summon panels (auxiliary)     │  │   probability                   │
│                                 │  │                                 │
│   "What's coming in"            │  │   "What I've decided"           │
│                                 │  │                                 │
└─────────────────────────────────┘  └─────────────────────────────────┘
```

The camera transition between screens is an animated horizontal slide
(~400ms ease-in-out). Player switches via a button or keyboard shortcut
(`TAB` or similar).

## Core gameplay loop

```
1.  Telemetry feed continuously streams fragments on SCREEN 1:
    [03:14:22] PAN-STARRS  // 2026-XB7 // a=1.42 e=0.61 i=4.2
    [03:14:31] ATLAS       // 2019-AQ3 // H=22.4
    [03:14:38] CATALINA    // 2026-XB7 // diameter=480m

2.  Player selects an active dossier (clicks an object_id from the
    dossier list, or types/picks it).

3.  Camera slides to SCREEN 2 (worksheet view). Active dossier shown
    with 4 empty field-group slots.

4.  Player switches back to SCREEN 1 (camera slides left), watches feed.

5.  When a relevant fragment arrives, player CLICKS the fragment in the
    feed → camera auto-slides to SCREEN 2 with the fragment "carried"
    → player CLICKS the appropriate slot to paste the fragment.

6.  Slot fills with fragment data. Worksheet shows accumulated data.

7.  Once enough slots are filled (minimum 2 of 4, but more = higher
    confidence), player issues a verdict: SAFE / MONITOR / DANGEROUS.

8.  Post-verdict reveal on SCREEN 2:
    - true_class
    - model_prediction + confidence
    - is_synthetic
    - whether player matched true_class
    - whether wrong-object fragments were pasted in
```

## The error mechanic (Variant A — fragment confusion)

The cognitive load lives in **step 5** above. Multiple ways the player
errs:

1. **Wrong object_id paste.** A fragment for `2019-AQ3` arrives, player
   intended to gather `2026-XB7` data, but the fragment "looks right"
   (numerical values similar), player pastes it. Worksheet now contains
   wrong-object data. Verdict will be computed against the *real
   target's* data + wrong-object's fragment.

2. **Wrong slot paste.** A `physical` fragment dropped into the
   `orbital` slot. Slot rejects type-mismatched fragments at v1 (visible
   error feedback). At v2+ can be made silent for higher difficulty.

3. **Conflicting fragments.** Two observatories report different
   `diameter` values for the same object. Player must choose which to
   trust, or flag conflict (this is a v2 feature, see Section 9).

The primary error source at v1 is **(1) wrong object_id paste**.

### What makes confusion possible

The backend pool must contain clusters of objects with **similar numerical
values in primary fields but differing in critical ones**. Specifically:

- Pairs of objects with close `a, e, i` but different MOID
- Pairs with close diameters but different orbital classes
- Pairs sharing 3 out of 4 doc-types' numerical ranges

This is enforced in `07_export.ipynb` as a sampling constraint when
selecting the 3000 game objects: prefer to include neighbours in feature
space, not maximally diverse points.

## Document types — recap

```
[DOC 1] ORBITAL SURVEY REPORT       — a, e, i, q, Q
[DOC 2] PROXIMITY ANALYSIS          — MOID, close approach date, rel_v
[DOC 3] PHYSICAL CHARACTERISATION   — H, diameter min, diameter max
[DOC 4] PROBABILITY UPDATE          — Palermo, Torino, impact_prob
```

## Difficulty progression

```
LEVEL 1 — ORIENTATION
─────────────────────────────────────────────────────
  • 1 active dossier at a time, feed slow (~1 fragment / 4s)
  • All 4 doc types delivered per object, in order
  • Soft timer (~3 min per object)
  • Mix: mostly real safe, 1–2 unambiguous real PHA
        to ensure player sees the DANGEROUS verdict at least once
  → Goal: player learns the screen-switching and copy-paste rhythm

LEVEL 2 — OPERATIONAL
─────────────────────────────────────────────────────
  • 2–3 active dossiers, feed normal speed (~1 fragment / 2s)
  • Fragments interleaved across objects
  • Per-object approach windows (90 sec each)
  • First synthetic borderline objects appear
  → Goal: first real mistakes — wrong-object fragment pastes

LEVEL 3 — CRISIS
─────────────────────────────────────────────────────
  • 5–7 dossiers, feed fast (~1 fragment / 1.2s)
  • Occasional REDACTED fields (MOID typically — "preliminary
    observation, MOID estimate pending refinement")
  • Approach window 60 sec
  • Mix: ~40% borderline synthetic in active threat pool
  → Goal: systemic overload, the Papers Please core experience

LEVEL 4 — EMERGENCY PROTOCOL (v2 feature)
─────────────────────────────────────────────────────
  • All of L3 plus:
  • Priority escalation pop-ups interrupting flow
  • Conflicting reports (two diameter values, must reconcile)
  • Mail-tab interruptions from NEOCC management
```

## Scoring & feedback

```
Per-verdict outcome lattice:

                  TRUE_CLASS:
                  SAFE     MONITOR   DANGEROUS
VERDICT: SAFE     correct  miss      CRITICAL MISS
       MONITOR    caution  correct   miss
       DANGEROUS  fp       caution   correct

A CRITICAL MISS (verdict=SAFE, true=DANGEROUS) is the worst outcome —
real-world equivalent of letting a planet-killer through. Tracked
separately. The score breakdown is shown after each shift, not after
each verdict (preserves immersion).

End-of-shift report:
   correct      :  17
   caution      :   5   (one step off)
   miss         :   2   (two steps off)
   CRITICAL     :   0   (planet-killers cleared as safe)
   wrong-fragment pastes detected: 3
```

`CRITICAL MISS` count > 0 should produce an in-universe mail-tab message
from NEOCC management in v2. For v1, just the score panel suffices.

---

# SECTION 7 — Frontend architecture reference

## Layout — hybrid windows model

```
┌────────────────────────────────────────────────────────────────────┐
│ HEADER — logo, session ID, dataset name, object counts, clock      │
├────────────────┬─────────────────────────────────────┬─────────────┤
│                │                                     │             │
│  TELEMETRY     │   MAIN VIEW                         │  RIGHT      │
│  FEED          │   (two-screen camera area)          │  PANEL      │
│                │                                     │             │
│  (fixed,       │   SCREEN 1: dossier list +          │  tabs:      │
│   continuously │             approach timeline       │  DOSSIERS   │
│   scrolling)   │   SCREEN 2: active worksheet        │  MAIL       │
│                │                                     │  HELP       │
│  Each line is  │   Camera slides horizontally        │             │
│  clickable     │   between them                      │             │
│                │                                     │             │
│                │   Summon-panels can appear on top   │             │
│                │   here as draggable floating elems  │             │
│                │                                     │             │
├────────────────┴─────────────────────────────────────┴─────────────┤
│ STATUS BAR — current status message, controls hint, audio toggle   │
└────────────────────────────────────────────────────────────────────┘
```

Concretely as CSS grid (echoing STELLaRUM's `220px 1fr 268px`):

```
#app {
  grid-template-columns: 240px 1fr 260px;
  grid-template-rows: 36px 1fr 28px;
}
```

## Fixed vs summonable panels

**Fixed (always present in the layout):**
- Header, status bar
- Telemetry feed (left column)
- Right panel with tabs (Dossiers list / Mail / Help)
- Main view area (the camera-driven two-screen region)

**Summonable (created on demand, draggable within main view):**
- Reference card ("what does Torino Scale 3 mean?")
- Prior cases lookup ("show similar past objects")
- Orbital diagram (v2)

Summon panels are floating DOM elements with `position: absolute` within
the main view region. Drag via plain `mousedown / mousemove / mouseup`
handlers, no library. Closing one returns its summon button to the panel
list. Maximum 3 summon panels open at a time (UX constraint, not technical).

**Mobile / touch is explicitly secondary.** This game is designed for
desktop with mouse and keyboard. Touch support exists as a courtesy for
the operator's friends with touchscreen laptops. Mobile phone playability
is not a goal.

## Camera transition

The main view contains two equally-sized children (`#screen-monitoring`,
`#screen-worksheet`) side by side in a flex container. The container is
translated horizontally to move between them:

```
.camera-container {
  display: flex;
  transition: transform 400ms cubic-bezier(0.4, 0, 0.2, 1);
}
.camera-container[data-screen="worksheet"] {
  transform: translateX(-100%);
}
```

Trigger on:
- Player clicks a fragment in feed → carry fragment + slide to worksheet
- Player clicks a slot on worksheet → paste fragment
- Player clicks "BACK TO MONITORING" button → slide to screen 1
- Keyboard shortcut (`TAB` or similar)

The carried fragment is shown as a small ghost element following the
cursor during the transition, snapping into the target slot on arrival.

## Telemetry feed

A vertical container fixed to the left column. New fragments append at
the bottom, container auto-scrolls down (auto-scroll pauses if the player
manually scrolls up). Each fragment is a row:

```
[03:14:22] PAN-STARRS  // 2026-XB7 // a=1.42 e=0.61 i=4.2
```

Each row carries a payload (`{source, object_id, doc_type, fields}`).
Clicking a row starts the fragment-carry interaction.

Feed length is capped (e.g., last 100 fragments visible) — older fragments
expire off the top. Once a fragment is consumed (clicked + pasted) it's
greyed out but stays visible briefly before being removed.

## State management

Vanilla JS module pattern, single global state object. No reactive
framework. No Redux. Echo the STELLaRUM approach:

```javascript
// ═══ STATE ═══
const state = {
  shift: { level: 1, startedAt: Date.now(), endsAt: null },
  pool:  [],            // loaded from game_objects.json
  feed:  [],            // active fragments in telemetry feed
  dossiers: new Map(),  // object_id → { fields: {...}, status: 'pending' }
  activeDossierId: null,
  cameraScreen: 'monitoring',  // or 'worksheet'
  carriedFragment: null,
  summonPanels: [],
  score: { correct: 0, caution: 0, miss: 0, critical: 0, wrongFragments: 0 }
};
```

Mutations are explicit function calls; renders are triggered manually
after state changes via small render functions per UI region.

## Sound

Web Audio API for procedural SFX, same approach as STELLaRUM:
- Fragment-arrival blip
- Fragment-click pickup
- Slot-paste confirm
- Verdict-submit
- Critical miss alarm
- Mail notification

Optional ambient track (`ambient.mp3`) faded in on first user interaction,
mirroring STELLaRUM's pattern.

## Splash screen

ASCII / SVG splash showing a stylised radio-telescope or NEO-monitoring
satellite, with project title in VT323, subtitle in Share Tech Mono, and
a blinking "PRESS ANY KEY" prompt. Same overall composition as STELLaRUM's
splash, different illustration content. Dismissable by click or any key.

---

# SECTION 8 — Phased execution plan

This is the working part of the document. Work through phases in order.
Each phase has:

- **Goal** — what this phase produces
- **Inputs** — what must exist before starting
- **Deliverables** — concrete files / artifacts created or modified
- **Steps** — ordered work items
- **Exit criterion** — objective signal that the phase is complete
- **Do NOT** — work that belongs to other phases, not this one

**At the end of each phase, pause and report to the operator. Wait for
explicit confirmation before starting the next phase.**

---

## PHASE 0 — Setup

**Goal:** A working project skeleton with directories, virtualenv, and
dataset downloaded.

**Inputs:** None.

**Deliverables:**
- Directory structure as per Section 3
- `backend/requirements.txt`
- `backend/.gitignore` (excludes `output/`, `*.pkl`, `*.npz`, `.ipynb_checkpoints`)
- Downloaded dataset CSV in `backend/data/raw/`
- Empty notebook files 01–07 with section headers only
- `frontend/planetary_defence.html` with just `<!DOCTYPE html>` and the
  STELLaRUM CSS variables block pasted in (no logic yet)

**Steps:**
1. Create directory tree per Section 3
2. Write `requirements.txt` with stack from Section 5
3. Set up `.gitignore` to keep notebook outputs out of git (configure
   `nbstripout` if available; otherwise just `.gitignore` the output dir)
4. Download dataset from Kaggle (operator may need to provide API token);
   place in `backend/data/raw/nasa.csv`
5. Create the seven notebook stubs with markdown headers but no code
6. Create the HTML stub with `<head>`, the palette CSS variables from
   Section 2, and an empty `<body>`

**Exit criterion:**
- `jupyter notebook` opens the seven empty notebooks without error
- Dataset CSV loads in a test cell (`pd.read_csv(...)` succeeds)
- HTML file opens in a browser showing a black page with the right font
  loading

**Do NOT:**
- Write any analysis code
- Write any game logic
- Implement the VHS overlay yet (Phase 7)

---

## PHASE 1 — EDA + Preprocessing

**Notebooks:** `01_eda.ipynb`, `02_preprocessing.ipynb`

**Goal:** Understand the dataset, clean it, produce train/test splits
saved as `.npz` artifacts ready for modelling.

**Inputs:** Phase 0 complete.

**Deliverables:**
- `01_eda.ipynb` filled with EDA cells + saved plots in `output/`
- `02_preprocessing.ipynb` filled with cleaning + split logic
- `output/dataset_full.npz` — X_train, X_test, y_train, y_test (full features)
- `output/feature_names.json` — column names in order for reproducibility

**Steps in `01_eda.ipynb`:**
1. Load raw CSV
2. Class distribution: bar chart + exact counts. Save
   `output/class_distribution.png`. Note the imbalance ratio explicitly
   in markdown.
3. Drop obvious garbage columns: `Name`, `Neo Reference ID` (after saving
   IDs separately for later game-data export), `Orbit Determination Date`,
   `Close Approach Date` if not used as feature, `Equinox`, `Orbiting Body`
4. Identify redundant features: multiple diameter units (km / m / miles /
   feet) — keep one. Show correlation matrix as evidence.
5. Distribution plots for key features: MOID, diameter, eccentricity,
   absolute magnitude, relative velocity. Hazardous vs non-hazardous
   overlay where useful. Save plots.
6. Pair plot on top 4–5 features coloured by hazardous flag.
7. Markdown summary at the end: how many objects, how many features after
   cleaning, how severe is the imbalance, which features look most
   discriminative by eye.

**Steps in `02_preprocessing.ipynb`:**
1. Load CSV, apply column drops from EDA decisions
2. Encode target: `Hazardous` boolean → 0/1
3. Stratified train/test split 80/20, `random_state=42`
4. Standardise features with `StandardScaler` (fit on train only,
   transform both)
5. Save `output/dataset_full.npz` with X_train, X_test, y_train, y_test,
   feature names
6. Save fitted scaler to `output/scaler.pkl` for later use

**Exit criterion:**
- Both notebooks run end-to-end without error
- Class distribution and pair plots saved to `output/`
- `dataset_full.npz` loads correctly in a fresh kernel and shapes match
  expectations

**Do NOT:**
- Train any models yet (Phase 2)
- Apply any resampling (Phase 3)
- Run MOID ablation (Phase 4)

---

## PHASE 2 — Baseline models

**Notebook:** `03_baseline_models.ipynb`

**Goal:** Train four models on the raw imbalanced data and document the
failure mode (high accuracy, poor hazardous recall).

**Inputs:** Phase 1 complete, `dataset_full.npz` exists.

**Deliverables:**
- `03_baseline_models.ipynb` complete with all four models + plots
- `output/metrics.json` initialised with baseline results
- `output/confusion_baseline_*.png` for each model

**Steps:**
1. Load `dataset_full.npz`
2. Train LogisticRegression, KNN(k=5), RandomForest(n_estimators=200,
   random_state=42), XGBoost. All with default class weights, no
   resampling.
3. For each model, compute and save:
   - accuracy
   - PR AUC (hazardous class) — **primary metric**
   - per-class precision, recall, F1
   - confusion matrix (save as PNG)
4. Build a comparison table (markdown) of all four models. Highlight:
   "all four hit 95%+ accuracy but hazardous recall is 20–50%. This is
   the failure mode we will address in notebook 04."
5. Initialise `output/metrics.json` with these baseline numbers, keyed
   under `"baseline"`.

**Exit criterion:**
- Notebook runs end-to-end clean
- All four confusion matrices saved
- `metrics.json` contains baseline section
- The "high accuracy, low hazardous recall" failure mode is clearly
  documented in notebook markdown

**Do NOT:**
- Apply SMOTE or any resampling (Phase 3)
- Tune hyperparameters extensively (use sensible defaults)
- Run feature ablation (Phase 4)

---

## PHASE 3 — Class imbalance handling

**Notebook:** `04_imbalance.ipynb`

**Goal:** Demonstrate three approaches to imbalance and pick the winner
empirically.

**Inputs:** Phase 2 complete, baseline RF results available.

**Deliverables:**
- `04_imbalance.ipynb` complete with three approaches compared
- `metrics.json` extended with `"imbalance"` section
- `output/confusion_imbalance_*.png` for each approach
- Decision recorded in markdown: which resampling strategy is the project's
  primary choice and why

**Steps:**
1. Load `dataset_full.npz`
2. Approach A: RF with `class_weight='balanced'`. Train, evaluate, save
   metrics + confusion.
3. Approach B: Borderline-SMOTE on training set, then RF (no class
   weight). Train, evaluate.
4. Approach C: ADASYN on training set, then RF. Train, evaluate.
5. Comparison table including baseline RF (no resampling) for context.
6. Plot: PR curves for all four (baseline + 3 approaches) on one chart.
   Save as `output/pr_curves_imbalance.png`.
7. Pick the winner based on **PR AUC on hazardous class**, with hazardous
   recall as tiebreaker. Document the choice in markdown.
8. Save the winning fitted model to `output/rf_full.pkl`.

**Exit criterion:**
- All three approaches evaluated with same metric set
- Winner selected and justified in markdown
- `rf_full.pkl` saved
- PR curves plot saved

**Do NOT:**
- Run MOID ablation (Phase 4)
- Generate synthetic objects for the game yet (Phase 5 — different
  purpose: SMOTE for *imbalance fix* here vs SMOTE for *game asset
  generation* in Phase 5; same algorithm, different scope and validation)

---

## PHASE 4 — MOID ablation experiment

**Notebook:** `05_moid_ablation.ipynb`

**Goal:** Show how much the model relies on MOID as a single feature.
Mirrors the SDSS redshift experiment from STELLaRUM.

**Inputs:** Phase 3 complete, winning RF config decided.

**Deliverables:**
- `05_moid_ablation.ipynb` complete
- `output/dataset_noMOID.npz` — train/test without MOID column
- `output/rf_noMOID.pkl` — model trained without MOID
- `metrics.json` extended with `"moid_ablation"` section
- `output/feature_importances.json` with both full and no-MOID importances
- `output/feature_importance_full.png` and `output/feature_importance_noMOID.png`

**Steps:**
1. Load `dataset_full.npz`, identify MOID column index
2. Create no-MOID version: drop column, re-fit scaler on train only,
   transform test. Save as `dataset_noMOID.npz`.
3. Train RF with the winning Phase 3 configuration on no-MOID data
4. Evaluate with same metric set as Phase 3. Save confusion matrices.
5. Extract feature importances from both models. Save as JSON and as
   two PNG bar charts (sorted descending).
6. Compute and report: by how much did accuracy / PR AUC / hazardous
   recall drop when MOID was removed? What share of full-model importance
   did MOID hold?
7. Markdown: physical interpretation — why MOID is so informative
   (it directly encodes the relevant question: "how close does this thing
   get to Earth?"). Two-three paragraphs, narrative writing for portfolio
   readers.

**Exit criterion:**
- Both models trained, both metric sets recorded
- Feature importances exported as JSON and PNG
- Drop in performance clearly quantified
- Physical interpretation written

**Do NOT:**
- Start synthetic generation (Phase 5)
- Begin export pipeline (Phase 6)

---

## PHASE 5 — Evil Model (synthetic borderline generation)

**Notebook:** `06_evil_model.ipynb`

**Goal:** Generate ~750 physically-valid synthetic borderline-hazardous
asteroids for the game pool. Implement the four-step physical validation
pipeline.

**Inputs:** Phase 4 complete, RF model + preprocessing pipeline available.

**Deliverables:**
- `06_evil_model.ipynb` complete with all four validation steps
- `output/synthetic_raw.json` — raw SMOTE output (pre-validation)
- `output/synthetic_validated.json` — after all four steps
- `output/synthetic_pool.json` — final 750 selected for the game
- `output/synthetic_validation_pairs.png` — pair plot real vs synthetic
- `output/synthetic_attrition.json` — counts at each validation stage

**Steps:**
1. Load preprocessed training data + winning RF
2. Identify INDEPENDENT physical features (`a, e, i, MOID, H,
   diameter_min, rel_v, albedo`) and DERIVED features (`q, Q,
   diameter_max`). Document this split in markdown.
3. Run Borderline-SMOTE on training data using only the independent
   features, generating ~1500 candidates (more than needed because of
   expected validation attrition).
4. Compute derived features for each candidate via physical formulas
   (q = a(1−e), Q = a(1+e), diameter_max from H+albedo).
5. **Step 1 validation:** apply hard clamps from Section 5. Drop
   violators. Record attrition.
6. **Step 2 validation:** convex hull / Delaunay membership check vs real
   NEO points in feature space. Drop outliers. Record attrition.
7. **Step 3 validation:** run the winning RF on validated candidates,
   keep only those where `0.45 ≤ confidence ≤ 0.75` (the borderline zone).
   This is what makes them "borderline" rather than just synthetic.
8. **Step 4 visual check:** pair plot of real PHA + validated synthetic
   on `(a, e, MOID, diameter)` axes. Save as PNG. Verify synthetic
   points lie among reals, not in a separate cloud.
9. Select final 750 synthetic objects (random sample from validated
   borderline set, seed 42). Save as `synthetic_pool.json`.
10. Markdown summary: started with X candidates, kept Y after clamps, Z
    after convex hull, W after confidence filter, sampled 750 for export.
    Note that the attrition rate (~50%) is itself a meaningful portfolio
    metric.

**Exit criterion:**
- All four validation steps implemented and documented
- Visual sanity-check plot saved and visually correct (synthetic among
  reals)
- 750 synthetic objects in final pool
- Attrition counts recorded

**Do NOT:**
- Build the game datapackage (Phase 6)
- Start frontend (Phase 7+)
- Generate fancy 3D plots or animations (not the deliverable here)

---

## PHASE 6 — Export pipeline (data contract)

**Notebook:** `07_export.ipynb`

**Goal:** Produce the final `game_objects.json` consumed by the frontend.
Implement the data contract from Section 4 exactly.

**Inputs:** Phase 5 complete, both real and synthetic pools available.

**Deliverables:**
- `07_export.ipynb` complete
- `output/game_objects.json` — final ~3000-object datapackage
- Copy of `game_objects.json` placed at `frontend/assets/data/`

**Steps:**
1. Load: real dataset (with original Neo Reference IDs), synthetic pool,
   trained RF, fitted scaler.
2. Run RF predictions on all real objects + all synthetic; record
   `model_prediction` and `model_confidence` per object.
3. Select pool composition: 250 real PHA + 750 synthetic borderline +
   2000 real safe (random sample). Total = 3000.
4. For each object, assemble the JSON structure exactly per Section 4
   data contract:
   - `object_id` (real ID for real objects; synthetic `YYYY-XX#` format
     for synthetic, year drawn from {2025–2032} for realism)
   - `is_synthetic` flag
   - `true_class` from dataset label (or "hazardous" for synthetic by
     construction)
   - `model_prediction` and `model_confidence` from RF
   - `fields` nested by doc-type group: orbital / proximity / physical /
     probability
   - `source_attribution` randomly assigned per object, seed 42
5. **Sampling constraint for player-confusion mechanic:** when selecting
   the 2000 real safe objects, prefer those near hazardous objects in
   feature space (not maximally diverse). This produces the
   "numerically similar but differently classed" clusters needed for the
   error mechanic. Use nearest-neighbour sampling: for each hazardous in
   the pool, include ~5 of its nearest non-hazardous neighbours by
   Euclidean distance in scaled feature space.
6. Validate JSON structure: every object has all required keys, no NaN
   or Infinity values, all numeric values are finite floats.
7. Write `output/game_objects.json` and copy to `frontend/assets/data/`.
8. Print summary stats: pool composition, model accuracy on this final
   pool, distribution of confidences for synthetic objects.

**Exit criterion:**
- `game_objects.json` exists, valid JSON, contains exactly 3000 objects
- Schema matches Section 4 exactly
- File copied to frontend assets directory
- Operator can manually inspect a few objects and confirm they look
  sensible

**Do NOT:**
- Start frontend implementation (Phase 7)
- Build the README (final phase)

---

## PHASE 7 — Frontend shell

**Goal:** Build the static shell of the frontend — visual identity, layout,
header, panels, status bar. No game logic yet, no data loading.

**Inputs:** Phase 6 complete, `game_objects.json` available in
`frontend/assets/data/`.

**Deliverables:**
- `frontend/planetary_defence.html` updated with:
  - Full CSS palette + typography from Section 2
  - Full VHS/CRT overlay (lifted from STELLaRUM as-is)
  - Three-zone grid layout: header, body row, status bar
  - Body row: telemetry feed column, main view, right panel with tabs
  - Tabs in right panel: DOSSIERS / MAIL / HELP (HELP populated with
    real controls reference; others placeholder)
  - Static dummy content in each zone showing the visual language works

**Steps:**
1. Lift STELLaRUM's `<head>` styles in full as a starting CSS block. Adapt
   layout grid to `240px 1fr 260px` for this project.
2. Lift the SVG chromatic-aberration filter and VHS overlay markup
   verbatim from STELLaRUM.
3. Build the header: logo (`PLANETARY DEFENCE OFFICER` in VT323, subtitle
   in Share Tech Mono), session ID, dataset name (`NASA NEO`), object
   count placeholder, clock.
4. Build the telemetry feed column: empty scrolling container, panel
   title `◈ TELEMETRY FEED`, placeholder text.
5. Build the main view: empty container with a clear visual frame, will
   later host the two-screen camera. For now show "MONITORING — no
   active session" placeholder.
6. Build the right panel with tab strip and three tab bodies. Implement
   tab switching (same pattern as STELLaRUM's `switchTab`).
7. Build the status bar: status message slot, controls hint, audio toggle
   button.
8. Fill the HELP tab with the controls reference from Section 7
   (keyboard shortcuts, what each zone does).

**Exit criterion:**
- HTML opens in browser, looks visually like a sibling of STELLaRUM
- VHS/CRT aesthetic active and visible
- All three zones plus header and status bar render correctly
- Tab switching works in the right panel
- No JavaScript errors in browser console
- HELP tab readable and informative

**Do NOT:**
- Load `game_objects.json` yet (Phase 8)
- Implement telemetry feed scrolling logic (Phase 8)
- Implement the two-screen camera (Phase 9)
- Build any game logic (Phases 8+)

---

## PHASE 8 — Telemetry feed + data loading

**Goal:** Load the game datapackage and stream fragments into the
telemetry feed. No interaction yet beyond visual flow.

**Inputs:** Phase 7 shell ready, `game_objects.json` accessible.

**Deliverables:**
- `planetary_defence.html` extended with:
  - State module (Section 7 state shape)
  - Data loader (`fetch('assets/data/game_objects.json')`)
  - Feed scheduler: emits one fragment every N seconds from a chosen pool
  - Feed renderer: appends rows to the telemetry feed column
  - Dossiers tab populates with active object_ids in the current pool

**Steps:**
1. Add the state object and basic state-update helpers.
2. On page load, fetch `game_objects.json`, parse, store in
   `state.pool`. Update header object count.
3. Build a deterministic shift initialiser: given a difficulty level,
   pick K active dossiers from the pool (K = 1 for L1, 2–3 for L2,
   5–7 for L3). Store in `state.dossiers`.
4. Build the fragment emission scheduler: at regular intervals, pick an
   active dossier, pick one of its four doc-types that hasn't been
   emitted yet, emit it as a feed row with source_attribution.
5. Render each fragment as a row in the feed column. Auto-scroll to
   bottom on new arrivals, but pause auto-scroll if user scrolls up.
6. Populate the DOSSIERS tab in the right panel with the active
   object_ids; clicking one selects it as the active dossier
   (`state.activeDossierId`). Selection is visual only at this phase
   (highlight). No camera switch yet.
7. Add a temporary debug control: a button "START L1 SHIFT" in the
   status bar that runs `startShift(1)`.

**Exit criterion:**
- Clicking the debug button starts a shift, fragments begin appearing
  in the feed every few seconds, feed scrolls
- DOSSIERS tab shows the active object IDs
- Clicking a dossier highlights it
- No console errors

**Do NOT:**
- Implement clicking on feed fragments (Phase 10)
- Build the worksheet / second screen (Phase 9)
- Build verdict logic (Phase 10)
- Tune difficulty timings precisely (Phase 11)

---

## PHASE 9 — Worksheet (screen 2) + camera transition

**Goal:** Build the second screen with empty field slots for the active
dossier. Implement camera slide between screens.

**Inputs:** Phase 8 complete, feed and dossiers working.

**Deliverables:**
- Main view restructured as `camera-container` with two children:
  `#screen-monitoring`, `#screen-worksheet`
- Worksheet UI with four empty field slots (orbital / proximity /
  physical / probability)
- Camera slide animation working (CSS transform)
- Keyboard shortcut (`TAB` or similar) and an explicit button to switch
  screens

**Steps:**
1. Restructure main view DOM: wrap existing monitoring content into
   `#screen-monitoring`, add `#screen-worksheet` as sibling, wrap both
   in `.camera-container`.
2. CSS: set up the camera container as `display: flex; width: 200%`
   with each child taking 50%. `transform: translateX(...)` controls
   which screen is active.
3. Add a function `switchCamera(target)` that updates
   `state.cameraScreen` and applies the transform.
4. Build the worksheet content: shows `state.activeDossierId` prominently
   at top, then four empty slots labelled ORBITAL / PROXIMITY /
   PHYSICAL / PROBABILITY. Each slot is a panel with a placeholder
   "awaiting data" state.
5. Add a BACK button on the worksheet that calls
   `switchCamera('monitoring')`. Add a SWITCH button or `TAB` keyboard
   shortcut on the monitoring screen.
6. If no active dossier is selected, the worksheet shows "NO ACTIVE
   DOSSIER — return to monitoring and select one".

**Exit criterion:**
- Two screens present in the DOM
- Camera slides smoothly (400ms ease) between them via button or
  keyboard
- Worksheet correctly shows the active dossier ID
- Worksheet shows four empty labelled slots
- No console errors

**Do NOT:**
- Implement copy-paste interaction (Phase 10)
- Implement verdict submission (Phase 10)
- Implement post-verdict reveal (Phase 10)
- Build summon-panels (Phase 12)

---

## PHASE 10 — Copy-paste + verdict flow

**Goal:** Wire up the core gameplay: clicking a fragment carries it,
clicking a slot pastes it, verdict button delivers the decision and
reveals the result.

**Inputs:** Phase 9 complete, both screens exist.

**Deliverables:**
- Clickable fragments in the telemetry feed
- "Carried fragment" visual indicator (ghost element following cursor or
  pinned to status bar — operator preference, default to status-bar
  banner for simplicity)
- Auto camera-switch to worksheet on fragment pickup
- Clickable empty slots that accept paste
- Filled slot UI showing the pasted data
- Verdict buttons (SAFE / MONITOR / DANGEROUS) below the slots
- Post-verdict reveal panel showing all outcome data per Section 6

**Steps:**
1. Make telemetry feed rows clickable. On click:
   - Set `state.carriedFragment` to the row's payload
   - Visual: row greys out, status bar shows "CARRYING:
     {object_id}/{doc_type}"
   - Camera auto-slides to worksheet
2. Make worksheet slots clickable when a fragment is carried. On click:
   - If `state.carriedFragment.doc_type` matches slot type AND
     `state.carriedFragment.object_id == state.activeDossierId`:
     paste succeeds, slot fills, fragment consumed
   - If `doc_type` matches but `object_id` does not: paste succeeds
     anyway (this is the error mechanic — wrong-object fragment is
     accepted because the slot only type-checks, not ID-checks).
     Record this in `state.dossiers[id].wrongFragments` for later
     scoring.
   - If `doc_type` does not match slot type: paste rejected with brief
     red-flash feedback. Fragment remains carried.
3. Render filled slots: show the actual numeric fields, source
   attribution, and a small indicator if the object_id mismatched.
   At v1, the mismatch indicator is HIDDEN from the player on the slot
   (revealed only after verdict). This preserves the cognitive challenge.
4. Add the three verdict buttons below the slots. Disabled until at
   least 2 slots filled.
5. On verdict click:
   - Look up the active dossier's true_class from the pool
   - Compute outcome via the lattice in Section 6
   - Update `state.score`
   - Show the post-verdict reveal panel (modal-style or replacing the
     worksheet content): true_class, model_prediction, model_confidence,
     is_synthetic, wrong-fragment count, verdict outcome category
6. After reveal acknowledged (player clicks "NEXT"), the dossier is
   removed from active set, a new one is drawn from the pool (or shift
   ends if pool exhausted).

**Exit criterion:**
- Player can complete a full loop: see fragments arrive, click one,
  paste into slot, submit verdict, see result, move to next dossier
- Score updates correctly across the outcome lattice
- Wrong-object paste is silently accepted at paste time, revealed at
  verdict time
- No console errors during a full L1 shift run

**Do NOT:**
- Implement approach window timers / time pressure (Phase 11)
- Implement levels 2+ specifics (Phase 11)
- Implement REDACTED fields (Phase 11)
- Polish sounds, mail, splash (Phase 12)

---

## PHASE 11 — Approach windows + difficulty levels

**Goal:** Add time pressure (per-dossier approach window timers) and
implement difficulty progression L1→L3.

**Inputs:** Phase 10 complete, core loop working.

**Deliverables:**
- Per-dossier countdown timer visible on the dossier list and worksheet
- Timer expiry triggers automatic verdict miss + outcome reveal
- Difficulty parameters configurable per level (number of active
  dossiers, feed speed, timer duration)
- REDACTED field rendering when applicable (occasional MOID hidden)
- Level-selection mechanism (basic — could be a status-bar dropdown or
  a tab in the right panel)

**Steps:**
1. Add `state.dossiers[id].approachWindow` with `startTime`, `duration`,
   `remaining`. Initialise per shift.
2. Per-frame update of remaining time (via `requestAnimationFrame` or
   `setInterval` — RAF preferred for smoothness).
3. Render timer on each dossier in the DOSSIERS tab and on the
   worksheet header for the active one. Visual: small progress bar +
   countdown text. Colour shifts from `--ok` to `--warn` to `--crit`
   as it approaches zero.
4. On timer expiry: if dossier still has no verdict, treat as missed —
   show outcome reveal with "TIMEOUT" framing, score as `critical` if
   true_class was DANGEROUS, else as `miss`.
5. Add level configuration object:
   ```javascript
   const LEVELS = {
     1: { dossiers: 1, feedInterval: 4000, approachWindow: 180000,
          redactedRate: 0, syntheticRate: 0 },
     2: { dossiers: 3, feedInterval: 2000, approachWindow: 90000,
          redactedRate: 0, syntheticRate: 0.2 },
     3: { dossiers: 7, feedInterval: 1200, approachWindow: 60000,
          redactedRate: 0.1, syntheticRate: 0.4 }
   };
   ```
6. Implement REDACTED field rendering: when a field is marked redacted,
   the value is replaced with `░░░░░░` in the feed AND in the slot if
   pasted. Note: redaction is applied at fragment-emission time, not
   in the source data.
7. Add level selector UI: a small dropdown or buttons in the status
   bar / debug area. Default to L1 on page load.

**Exit criterion:**
- Selecting L1, L2, or L3 starts a shift with the corresponding
  parameters
- Per-dossier timers run, expire correctly, trigger timeout outcome
- L3 produces visible REDACTED fields occasionally
- Synthetic objects appear at the rates configured per level
- A full shift can be played L1→L3 progressively

**Do NOT:**
- Implement L4 / emergency protocol (v2 feature)
- Implement conflict resolution (v2 feature)
- Implement summon-panels (Phase 12 if pursued, else v2)
- Implement multi-shift / persistent campaign mode (v2)

---

## PHASE 12 — Polish

**Goal:** Add the finishing layers — splash screen, sound, mail-tab
narrative messages, summon-panels (if pursued in v1).

**Inputs:** Phase 11 complete, core game playable end-to-end.

**Deliverables:**
- Splash screen on first load, dismissable
- Procedural Web Audio sound effects for key events
- Optional ambient track support (file may or may not exist)
- Mail tab with at least 3–5 narrative messages delivered at score
  milestones
- Summon-panel system: at least one panel (Reference Card for risk
  scales) demonstrating the pattern
- Final visual pass on spacing, typography, colour consistency

**Steps:**
1. Splash: SVG illustration appropriate to the project (telescope dish,
   asteroid silhouette, ESA NEOCC-style emblem — pick one with the
   operator). Project title in VT323, subtitle, blinking prompt.
   Dismiss on click or any key.
2. Web Audio: copy STELLaRUM's `playTone` helper, add procedural sounds
   for: fragment arrival (brief blip), fragment pickup, slot paste,
   verdict submit (three variants — SAFE soft chime, MONITOR neutral,
   DANGEROUS low buzz), CRITICAL MISS alarm.
3. Mail tab: implement the same inbox pattern as STELLaRUM. Trigger
   messages at: first session start (welcome), first correct DANGEROUS
   verdict (commendation), first CRITICAL MISS (warning), end of shift
   (performance summary).
4. Summon-panels (if pursued in v1, otherwise defer):
   - Create the floating-panel base class: absolute-positioned, header
     bar with drag handle, content area, close button
   - Implement plain JS drag (mousedown → mousemove → mouseup), no
     library, no resize at v1
   - Add a Reference Card summon: explains Torino, Palermo, MOID
     thresholds
   - Add a button to summon it from the worksheet header
5. Final visual pass: verify every panel, button, label uses the
   palette CSS variables, no hardcoded colours; verify monospace
   consistency; verify VHS overlay doesn't fight any UI element
   visually.

**Exit criterion:**
- Splash appears on first load and dismisses correctly
- Audio feedback present for major actions; can be toggled
- Mail tab has populated content delivered through gameplay
- Summon-panel system works for at least one panel (if pursued)
- Project plays end-to-end as a polished v1

**Do NOT:**
- Implement orbital diagram view (v2)
- Implement ONNX live inference (v2 / wishlist)
- Implement Level 4 emergency protocol (v2)

---

# SECTION 8.1 — Milestone v1.1: Operator Console

> **Status:** planned 2026-06-22, after v1 (Phases 0–12) shipped and was
> browser-verified. This milestone is a deliberate re-vision of the *feel*
> of the core loop, not a rewrite of its mechanics.

## Why this milestone

v1 is functionally complete but visually "raw": the main view is sparse and
the worksheet reads as a tidy single-column form. That tidiness works against
the original design intent. The fantasy is a **real operator's console** — a
cluttered, slightly overwhelming "chest-of-screens" surface where information
for a single verdict is scattered across **several non-parallel source
windows** that the officer must gather, cross-check, and reconcile. The
cognitive load of *managing the surface itself* is part of the difficulty: it
raises the human-factor error rate that the SAFE/MONITOR/DANGEROUS judgement
depends on. v1.1 builds that surface.

This promotes two items previously deferred to v2 (see Section 9): the
**fully-floating workstation variant** and **conflict resolution / misleading
sources**. The draggable Reference Card summon-panel shipped in Phase 12 is the
first building block of this paradigm.

## Leading assumption (confirm before executing Phase 15+)

The floating-windows surface is treated as an **evolution layered on top of**
the existing telemetry feed + camera + worksheet, **not** a replacement of the
feed. The feed remains the canonical Variant-a fragment stream; the worksheet's
fixed slots evolve into draggable source/dossier windows. If the intent is
instead to retire the feed/worksheet split entirely, Phases 15–16 need a
different decomposition — settle this in a discuss pass first.

## Ordering

Cheap atmosphere and motion first (13–14, low risk, no design ambiguity), then
the paradigm work (15–16, design-heavy — discuss before executing), then the
final consistency pass (17). One phase at a time; pause for review at each
boundary, as in Section 8.

---

## PHASE 13 — Atmosphere layer

**Goal:** Make the console feel alive and powered-on, matching STELLaRUM's
ambient presence, without touching game logic.

**Inputs:** v1 complete.

**Deliverables:**
- Animated canvas background behind the main view (drifting starfield /
  low-amplitude noise / occasional sweep), echoing STELLaRUM's `drawNoise`.
- Ambient audio bed (low hum / room tone) gated by the existing audio toggle,
  layered under the procedural `sfx.*` from Phase 12.
- Background sits below all UI and the VHS overlay; never intercepts clicks.

**Steps:**
1. Add a `<canvas>` fixed behind `#app` (or inside the main view), z-index
   below content. Render loop via `requestAnimationFrame`, throttled.
2. Port STELLaRUM's noise/starfield pattern; tune density to the green/grey
   palette so it reads as "instrument glow," not snow.
3. Add an ambient `OscillatorNode`/buffer hum through a low gain into the
   existing `audioCtx`; start on audio-enable, stop on disable.

**Exit criterion:**
- Background animates smoothly at a low CPU cost; toggling audio starts/stops
  the hum; no interaction is blocked; no console errors.

**Do NOT:**
- Add gameplay-affecting visuals (timers, data) to the background — that is
  Phase 15's MONITORING work.

---

## PHASE 14 — Motion & micro-interactions

**Goal:** Add the tactile feedback layer that sells the interface.

**Inputs:** Phase 13 complete.

**Deliverables:**
- Carried-fragment "ghost" element that follows the cursor between feed and
  worksheet (specified in Section 7, not yet built), snapping into the slot.
- Fragment-arrival animation in the feed (fade/slide-in), paste/consume
  animation, type-mismatch shake already exists — unify timings.
- Camera-slide and reveal-panel transitions polished (easing, no jank).
- Respect `prefers-reduced-motion`.

**Exit criterion:**
- Carry → paste reads as continuous motion; reveal panel animates in/out;
  reduced-motion users get instant states; clean console.

**Do NOT:**
- Introduce new windows or change layout structure (Phase 15).

---

## PHASE 15 — Floating workstation surface

**Goal:** Replace the tidy fixed worksheet column with a draggable,
overlapping **window surface** — the cluttered operator's desk.

**Inputs:** Phase 14 complete; **discuss pass done** confirming the leading
assumption above.

**Deliverables:**
- A reusable window-manager generalising the Phase 12 summon-panel base:
  create / focus (z-order) / drag / close, with the Section 7 cap (max ~3–5
  open). Single source of truth in `state.summonPanels`.
- The active dossier's source material rendered as **separate draggable
  windows** (one per source/doc-type, or per the discuss outcome) instead of
  fixed slots — the player arranges them on the desk.
- Verdict controls remain reachable but the surface is intentionally dense.

**Open design questions (resolve in discuss):**
- One window per observatory, per doc-type, or per dossier?
- Do windows spawn automatically on dossier-open, or are they summoned?
- How does "fill ≥2 slots to verdict" map onto windows?
- Snap-to-grid vs. free placement; do positions persist per dossier?

**Exit criterion:**
- A dossier can be worked entirely through floating windows: gather → arrange
  → reconcile → verdict; windows drag/focus/close; cap enforced; clean console.

**Do NOT:**
- Build the misleading/conflicting-source logic yet (Phase 16).

---

## PHASE 16 — Multi-source verdict & deliberate misdirection

**Goal:** Deepen the Variant-a error mechanic: a verdict is composed of
several sub-judgements, each sourced from a **non-parallel** window, and
sources can conflict or mislead.

**Inputs:** Phase 15 complete.

**Deliverables:**
- Verdict decomposed into named sub-items (e.g. proximity risk, size,
  impact probability) each resolved from its own source window.
- Conflicting reports: two sources disagree on a field; the player must pick
  which to trust or flag the conflict (promotes the v2 "conflict resolution"
  item).
- Misdirection surfaces tuned to the level system (more conflicts / near-
  duplicate values at higher levels), reusing the existing wrong-object
  fragment scoring.

**Open design questions (resolve in discuss):**
- Is the final verdict still a single SAFE/MONITOR/DANGEROUS, or a composite
  scored across sub-items?
- How is a "correct reconciliation" defined and scored vs. the current model?

**Exit criterion:**
- A full dossier requires consulting ≥2 non-parallel windows; at least one
  level surfaces a genuine conflict the player must resolve; scoring reflects
  reconciliation quality; clean console.

**Do NOT:**
- Add Level 4 / emergency-protocol interrupts (still v2).

---

## PHASE 17 — Typography & visual consistency pass

**Goal:** Lift the overall finish to STELLaRUM's refinement.

**Inputs:** Phase 16 complete.

**Deliverables:**
- Spacing/rhythm audit across header, panels, windows, status bar; consistent
  type scale and letter-spacing; every colour from the palette variables.
- Window chrome, shadows, and borders unified; VHS overlay verified not to
  fight any new floating element.
- Final pass screenshot comparison against STELLaRUM.

**Exit criterion:**
- Side-by-side with STELLaRUM, Planetary reads as the same series at the same
  polish tier; no off-palette colours; clean console.

**Do NOT:**
- Start v2 features (orbital diagram, L4, campaign mode).

---

# SECTION 9 — Scope split: v1 / v2 / wishlist

This split is **binding**. Do not implement v2 or wishlist features
during v1 phases without explicit operator approval, regardless of how
small or easy they seem.

## v1 (in-scope, Phases 0–12)

- Full backend ML pipeline (notebooks 01–07)
- Borderline-SMOTE evil model with physical validation
- Single-file HTML frontend, vanilla JS
- Two-screen camera (monitoring / worksheet)
- Telemetry feed with fragment-click → paste interaction
- Wrong-object paste error mechanic (Variant A)
- Approach-window timers
- Levels 1, 2, 3
- Three verdict types (SAFE / MONITOR / DANGEROUS)
- Mail tab with milestone messages
- VHS/CRT visual aesthetic from STELLaRUM
- Splash screen
- Procedural Web Audio sound effects
- At least one summon-panel (Reference Card)

## v2 (post-v1, explicitly deferred)

- 2D orbital diagram view as a summon-panel. Shows the active object's
  orbit relative to Earth's, with `a`, `e`, `i` driving the ellipse
  geometry. Important bridge feature toward future projects in the
  portfolio series. Re-uses Canvas patterns from STELLaRUM.
- Level 4 (emergency protocol): priority escalation interrupts,
  conflicting reports, mail interruptions
- Conflict resolution mechanic: two observatories disagree on a
  field, player picks which to trust or flags conflict
  — **PROMOTED to milestone v1.1, Phase 16 (Section 8.1)**
- Multi-shift / campaign mode with persistent score across shifts
- Movable / resizable dossier windows (the fully-floating workstation
  variant — see Section 7)
  — **PROMOTED to milestone v1.1, Phase 15 (Section 8.1)**
- ADASYN as primary resampling if it wins Phase 3 ablation by a clear
  margin
- Mobile / touch optimisation pass

## Wishlist (may never happen, and that's fine)

- ONNX in-browser live inference: player can slide a feature value
  and watch the RF prediction update in real time
- Procedurally generated NEO surveys based on real orbital element
  distributions
- Multilingual UI (currently English-only, with Russian as a stretch
  if the operator wants it)
- Voice / audio briefings from "NEOCC director" at shift starts

---

# SECTION 10 — Working rules for the agent

## Code style

- **Vanilla JS only.** No React, no Vue, no Svelte, no jQuery, no
  Tailwind, no build step. The frontend is one HTML file plus a CDN
  font import.
- **Section markers in JS.** Echo STELLaRUM's pattern: divide the
  script body into clearly labelled sections with `// ═══...═══` headers
  (STATE, CANVAS SETUP, RENDERING, INPUT, GAME LOGIC, AUDIO, etc.).
  Search for these in `stella_rum.html` to see the convention.
- **Single global `state` object** rather than scattered let-variables.
  Mutations are explicit function calls.
- **No async-await sprawl** in game loop code — keep timing logic in
  `requestAnimationFrame` and `setInterval`, fetch only for the initial
  JSON load.
- **CSS variables for all colours.** No hardcoded hex values in
  component-specific CSS. If a new colour is needed, add it as a
  variable.
- **Monospace everywhere.** If you find yourself wanting a different
  font for emphasis, use VT323 (the title font) or weight/size variation
  instead.

## Notebook hygiene

- **Strip outputs before commit.** Either use `nbstripout` as a git
  hook or manually clear outputs in Jupyter before staging. Embedded
  PNG outputs in notebook JSON make diffs unreadable. Save plots
  separately to `output/*.png` and reference them in the README rather
  than relying on embedded outputs.
- **Each notebook self-contained.** Loads its inputs from `output/`
  (artifacts produced by earlier notebooks), saves its outputs to
  `output/`. Should run end-to-end from a fresh kernel.
- **Markdown narration.** Every notebook tells a story. Markdown cells
  introduce each section ("Now we test whether removing MOID
  collapses model performance"), summarise findings ("Removing MOID
  dropped hazardous recall from X to Y"). Code-only notebooks are
  hard to read as portfolio pieces.

## Reproducibility

- `random_state = 42` everywhere it can be set: train/test split, RF
  initialiser, KNN where applicable, SMOTE generators, the
  source_attribution randomisation in Phase 6, the synthetic pool
  sampling in Phase 5.
- Document seed usage in the markdown of each notebook that uses
  randomness.

## Metric discipline

- **Never report only accuracy on imbalanced data.** Always include
  PR AUC for the hazardous class, per-class precision/recall/F1, and a
  confusion matrix plot.
- Save metric tables to `output/metrics.json` cumulatively. By Phase 6
  the JSON should contain a full audit trail: baseline, imbalance
  experiments, MOID ablation, synthetic-generation impact.

## Synthetic generation discipline

- Always apply the four-step physical validation pipeline from Section 5.
- Never skip the visual pair-plot check. If synthetic looks visually
  separate from real, stop and debug.
- Treat attrition rates as a portfolio metric — they show methodological
  awareness, not failure.

## Per-phase discipline

- One phase at a time. Read the phase's "Do NOT" list before starting,
  not after.
- Report phase completion to the operator with:
  - what was done
  - exit criterion satisfaction
  - any deviations or questions
- Wait for operator confirmation before starting the next phase.

## When stuck

- Re-read the relevant reference section
- Check `stella_rum.html` for the pattern (most frontend patterns
  already exist there)
- Ask the operator a focused question rather than guessing
- For dataset / Kaggle access issues, defer to operator (the operator
  has the API token)

---

# SECTION 11 — Open questions

The following design points are deliberately left unresolved at v1
spec time. They should be raised with the operator at the relevant
phase, not silently decided by the agent.

1. **Specific narrative voice for mail-tab messages.** STELLaRUM's mail
   tab had a specific tone (terse, formal, light flavour). The
   planetary defence equivalent — messages from NEOCC management,
   ESA director, etc. — needs a tone decision. Raise in Phase 12.

2. **Splash screen illustration content.** A telescope dish? A NEO
   silhouette over Earth? A NEOCC operations centre stylised
   schematic? Pick with the operator in Phase 12.

3. **Difficulty timing exact values.** L1/L2/L3 feedInterval and
   approachWindow values in Section 8 / Phase 11 are starting points,
   tunable after playtest. Confirm with operator after first
   playable build.

4. **Synthetic object ID format.** Real objects use
   `YYYY-XX#` Minor Planet Center provisional designation. Synthetic
   objects need to either follow this format (potentially confusing
   players) or be visibly different (potentially breaking immersion).
   Default: follow the format with reserved year prefix (e.g.,
   `2099-XX#` for synthetic), but defer the final choice to operator.

5. **Hazardous-pool ratio (33%).** Current spec is 1000 hazardous /
   2000 safe in the game pool. This may be tuned after playtest.
   Raise after first L3 shift is playable.

6. **Should pasted-fragment object_id mismatch be visible immediately
   on the worksheet, or revealed only at verdict?** Current spec: hidden
   until verdict, for maximum cognitive challenge. May be too punishing.
   Tunable.

7. **Ambient audio track.** If used, what kind? STELLaRUM had a
   specific track that fit. Planetary defence might want something
   different (more tension, more procedural). Operator decision.

---

```
╔══════════════════════════════════════════════════════════════════╗
║  END OF AGENT WORKING DOCUMENT                                   ║
║  PLANETARY DEFENCE OFFICER — v1.0                                ║
║  Proceed phase by phase. When in doubt, ask.                     ║
╚══════════════════════════════════════════════════════════════════╝
```
