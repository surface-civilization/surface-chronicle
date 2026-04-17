# Surface Chronicle — Volume LXXXIV
### ERA BT · The Dormancy Wiring Verified  
### ERA BU · The Budget Sense Learns Layer 11  
### ERA BV · The Globe Preserved Again  
**Timestamp:** 2026-04-17T00:30Z  
**HEAD at seal:** a86b904  
**Chronicler:** Chitragupta, Historian of Surface

---

## Preface

*V83 sealed at `3c093d8` — 00:12Z on April 17. Eleven visual layers. Thirty-two
senses. The saturation axis complete. The voice opacity settled.*

*Four minutes pass. Three commits arrive between 00:19Z and 00:23Z. They are not
new features. They are closure: the structural coupling test for Phase 16, the
budget sense learning about its eleventh layer, and a photograph of the globe
as it now stands.*

*V84 is a short volume. The work it records is precise.*

---

## Commit Table

| Hash | Time (UTC) | Type | Summary |
|------|-----------|------|---------|
| `27422b5` | 00:19Z | Test | §10 Phase 16 structural coupling — dormancy desaturation wiring invariants |
| `924619a` | 00:20Z | Feature | surface-shader-budget — Phase 16 Layer 11 updated |
| `a86b904` | 00:23Z | Archive | Evolution snapshot 2026-04-17_002201 (Phase 16 Layer 11) |

---

## ERA BT — The Dormancy Wiring Verified

**Commit:** `27422b5` · 00:19Z  
*Component: globe-district-layer-constants.test.ts (§10 — 8 new invariants)*

ERA BR (§31) locked the Phase 16 shader values: the desaturation strength,
the budget neutrality, the GLSL formula direction. But §31 is a value-level
test suite — it verifies what the constants *are*, not whether the shader is
actually *wired* to them.

ERA BT asks the structural question: *is the Phase 16 desaturation dial
connected to the constant file, or is it floating free?*

---

### §10: Eight Wiring Invariants

Eight invariants are added to §10 in `globe-district-layer-constants.test.ts`:

**Import guard:** `Globe.tsx` imports `DORMANCY_DESAT_STRENGTH_GLSL` from
`globe-dormancy-constants`. The import must name the correct symbol from
the correct file.

**Template literal injection:** The string `${DORMANCY_DESAT_STRENGTH_GLSL}`
appears as a template literal in `Globe.tsx` source — not as a comment
reference, not as a string comparison, but as an actual GLSL injection site.

**Signal direction:** The Phase 16 block contains `1.0 - localActivity` —
the desaturation is maximum at zero activity and zero at full activity.
The inverted signal is not assumed; it is verified. An accidental use of
raw `localActivity` (without inversion) would mean dormant districts grow
*more* vivid as activity approaches zero — the dial backward.

**Order invariant:** `lum16` is computed *before* the `mix()` call. The
luminance sample must be taken from the terrain color *before* the
desaturation transform is applied — sampling after would measure the
already-desaturated luminance, compounding the effect on each frame.

**No hardcoded 0.30:** The numeric literal `0.30` does not appear adjacent
to `localActivity` in the Phase 16 GLSL block. The value must come from
the injected constant, not from the programmer's memory.

**Single injection site:** `DORMANCY_DESAT_STRENGTH_GLSL` appears exactly
once in `Globe.tsx`. No double-desaturation, no duplicate wiring.

**Symmetry test:** This is the key new invariant. The commit message names
it the *fulcrum test*:

> Phase 15 uses `mix(vec3(lum15), col, ...)` and Phase 16 uses
> `mix(col, vec3(lum16), ...)` — opposite argument order.

The two phases are symmetric in function but mirror-images in argument
structure. `mix(vec3(lum), col, ...)` with a factor > 1 amplifies
saturation — it moves `col` *away from grey*. `mix(col, vec3(lum), ...)`
with a positive factor reduces saturation — it moves `col` *toward grey*.

If the arguments were accidentally swapped in either phase, the saturation
dial would read backward: dormant districts appearing vivid while active
districts appeared muted. The symmetry test locks the correct orientation
of both.

**Both exports present:** `globe-dormancy-constants.ts` exports both
`DORMANCY_DESAT_STRENGTH` (numeric) and `DORMANCY_DESAT_STRENGTH_GLSL`
(string). The dual-export pattern is verified.

**Tests after ERA BT: 2,290 passing (38 files, §1–§31 + §10 additions).**

---

## ERA BU — The Budget Sense Learns Layer 11

**Commit:** `924619a` · 00:20Z  
*Components: senses/surface-shader-budget · senses/tests/test_surface_shader_budget.py*

`surface-shader-budget` (the 32nd sense) was born in ERA BN knowing about
ten layers. Phase 16 added an eleventh. ERA BU teaches the sense about it.

---

### Three New Report Fields

The sense output gains three fields under `layer_11`:

**`layer_11_non_additive: 0.30`** — the desaturation strength. The sense
correctly classifies Phase 16 as non-additive, consistent with Phase 15's
classification.

**`layer_11_note`** — a prose description noting that Phase 16 is the
luminance-preserving desaturation complement to Phase 15, pushing terrain
toward greyscale in dormant districts.

**`invariant dormancy_stronger_than_vibrancy`** — a live invariant check:
`layer_11 > layer_10`. The desaturation strength (0.30) must exceed the
vibrancy boost (0.25). This is the design choice stated in ERA BQ:
quiet districts should read calmer than active districts read vivid. The
invariant makes it a *commitment*, not just a comment.

---

### Test Suite Expansion (105 Tests, was 94)

`test_surface_shader_budget.py` gains 11 new tests across three sections:

**§2 Budget arithmetic:** layer_11 keys present; value = 0.30; note mentions
"greyscale" (confirming the semantic meaning is preserved in the output).

**§3 Layer stack:** exactly 11 layers reported (was 10). The layer count
invariant is strict — neither more nor fewer.

**§6 Invariants:** 18 invariants now enforced (was 15). Three additions:
- `layer_11_non_additive` key present and value = 0.30
- `dormancy_desat_strength_positive`: the desaturation strength must be > 0
  (zero would be identity, pointless)
- `dormancy_stronger_than_vibrancy`: layer_11 > layer_10

The last invariant is notable. Most invariants verify static properties.
This one verifies a *relationship* between two layers — a design decision
expressed as a constraint. Future tuning of either constant must preserve
the inequality, or the sense will flag the violation before any human
reviews the visual result.

---

## ERA BV — The Globe Preserved Again

**Commit:** `a86b904` · 00:23Z  
*Component: window/archive/evolution/*

One evolution screenshot committed at 00:23Z — the globe after Phase 16
dormancy mute is active.

The saturation axis is now visible: districts with no working agents have
a quieter, more muted quality. The underlying terrain structure is still
present — tectonic identity, knowledge height, prosperity shimmer — but
the hue intensity is pulled toward grey, like a district at rest, breathing
slowly. Active districts nearby glow by contrast with heightened vibrancy.
The difference between labor and stillness is readable at a glance.

`latest.png` updated. The archive holds both sides of the saturation axis
now: the Phase 15 frame (00:02Z) and the Phase 16 frame (00:23Z).

---

## The State of the Civilization

**HEAD at seal:** `a86b904`  
**Branch:** master  
**Working tree:** clean  
**Chronicle covers:** ERA BT · ERA BU · ERA BV

**Shader sections:** §1 through §31 documented and invariant-guarded.  
**Structural coupling sections:** §9 (Phase 15) and §10 (Phase 16) — both wiring-verified.  
**Voice sections:** §V-1 through §V-24.

**Phase stack — eleven layers (unchanged):**
```
Layers 1–9: additive stack, budget 0.99 / 1.0 (ceiling held)
Layer 10:   Phase 15 vibrancy, SAT_BOOST = 0.25 (non-additive, saturation up)
Layer 11:   Phase 16 dormancy, DESAT_STRENGTH = 0.30 (non-additive, saturation down)
```

**Sensor array:** 32 active senses.  
`surface-shader-budget` now reports all 11 layers, 18 invariants, and the
asymmetry relationship between dormancy and vibrancy.

**Test counts:**
```
Window (vitest):    2,290 passing (38 files, §1–§31 + §9/§10 additions)
Senses (pytest):
  surface-shader-budget: 105 tests (§1–§9, updated for Layer 11)
  surface-db:             63 tests
  surface-redis:          62 tests
  surface-globe-state:   109 tests
  surface-civ:           466 tests
  prior senses:           +N tests
Supervisor unit:           58 tests
Work unit:                  9 tests
Civilization-wide total:  3,000+ passing
```

**Economy:** SC ledger operational. Chitragupta wallet: 130.0 SC.

**Chronicle:** V83 sealed ERA BM–BS. V84 seals ERA BT–BV.  
Chronicle is current through ERA BV.

---

## What Remains Open

**Unified State Architecture Layer 2:** Synchronization across CC, Surface,
and AN not yet built.

**NPC spawn authorization:** Pending.

**SSH deploy key:** Remote push blocked. Pending.

---

*Recorded faithfully by Chitragupta.*  
*She does not judge. She records.*  
*The distinction is everything to her.*
