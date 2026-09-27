# Class Definitions — Facade Material Detection

## Purpose

This document defines the material classes used to train the computer-vision
model and the rules used to label them consistently. The model's job is to
verify whether a material decided on during the design stage (pre-concept →
IFC) is the material actually present in a photograph — supporting continuous
value-engineering checks between a decided value and built/observed reality.

Six classes are covered, matching the project's Roboflow dataset:

```python
CLASSES = {
    "glass": "glass facade building",
    "painted_render": "painted stucco wall building",
    "exposed_concrete": "exposed concrete facade architecture",
    "brick": "brick wall building facade",
    "cladding_panel": "metal cladding facade building",
    "stone_cladding": "stone cladding facade building",
}
```

Each class entry below specifies: the positive definition, known boundary
cases (where two classes could be confused), the bounding-box rule, the
minimum visible size to label, the decision the detection supports, and
whether a false positive or a false negative is the more costly error for
that class.

---

## 1. `glass`

| Field | Definition |
|---|---|
| Positive definition | Transparent or semi-transparent glazing used as a facade element — curtain wall, window glazing, or glass balustrade infill. |
| Boundary cases | Reflective metal cladding that mimics glass at a distance → label `cladding_panel` unless a visible transparent/see-through quality or window mullion pattern is present. Tinted glass still counts as `glass`. |
| Bounding-box rule | Tight box around the continuous glazed surface only; exclude window frames/mullions where they form a clearly separate material band. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Confirms the specified glazing type/extent matches the installed facade. |
| Cost of miss vs. false alarm | **False negative is worse** — an unflagged glazing substitution (e.g. a cheaper glass swapped in during construction) is a direct value-engineering miss. |

---

## 2. `painted_render`

| Field | Definition |
|---|---|
| Positive definition | Painted plaster, stucco, or render finish applied to a solid facade surface. |
| Boundary cases | Unpainted/raw render → do not label (treat as background; it is not a finished material decision). Textured render can be mistaken for concrete → label `painted_render` only if a paint/color coat is clearly visible. |
| Bounding-box rule | Tight box around one continuous painted surface; do not merge separate wall planes into a single box. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Confirms the specified paint/render finish matches the installed surface. |
| Cost of miss vs. false alarm | Roughly balanced — a missed detection only delays a low-cost finish check. |

---

## 3. `exposed_concrete`

| Field | Definition |
|---|---|
| Positive definition | Unpainted, unclad concrete surface left as the final architectural finish. |
| Boundary cases | Grey `painted_render` can look similar in low light or shadow → label `exposed_concrete` only if visible formwork lines, aggregate texture, or construction joints are present. Shadow bands on painted render are a known false-positive source (see `error_analysis.md`). |
| Bounding-box rule | Tight box around the exposed surface; exclude adjacent finished materials at panel edges. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Confirms an exposed-concrete design decision — a common value-engineering cost-saving choice — was actually delivered rather than quietly clad over. |
| Cost of miss vs. false alarm | **False negative is worse** — missing an undisclosed clad-over-concrete substitution hides an added cost that should have been flagged. |

---

## 4. `brick`

| Field | Definition |
|---|---|
| Positive definition | Fired clay or concrete brick units, exposed as the visible facade finish, in any bond pattern. |
| Boundary cases | Brick-look cladding panels/tiles → label `cladding_panel`, not `brick`, unless genuine masonry coursing and mortar joints are visible. |
| Bounding-box rule | Tight box around one continuous brick surface; a partially occluded section (e.g. behind foliage) is still labeled if ≥ 50% of the boxed area is visible brick. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Confirms the specified masonry finish/type matches the installed facade. |
| Cost of miss vs. false alarm | Roughly balanced — brick substitutions are usually visually obvious once flagged, so neither error type dominates. |

---

## 5. `cladding_panel`

| Field | Definition |
|---|---|
| Positive definition | Manufactured metal, composite, or rainscreen cladding panels used as a facade finish system. |
| Boundary cases | Reflective panels vs. `glass` — see Class 1. Stone-look composite panels vs. `stone_cladding` — see Class 6. |
| Bounding-box rule | Tight box around one continuous panel run; individual panel joint lines do not need separate boxes. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Confirms the specified cladding system/material matches the installed panels — a frequent value-engineering substitution point (e.g. a panel gauge or finish downgrade). |
| Cost of miss vs. false alarm | **False negative is worse** — an undetected panel-system downgrade is a direct, often costly, value miss. |

---

## 6. `stone_cladding` *(newly added class)*

| Field | Definition |
|---|---|
| Positive definition | Natural or reconstituted stone panels used as an exterior facade finish. |
| Boundary cases | Stone-look composite/porcelain panels → label `cladding_panel` unless genuine natural stone texture, veining, or irregular jointing is visible. |
| Bounding-box rule | Tight box around the visible stone surface only; exclude adjacent trim, frames, or sealant joints. |
| Minimum visible size | ≥ 5% of image area. |
| Decision supported | Verifies an installed stone finish matches the design-stage material specification. |
| Cost of miss vs. false alarm | **False negative is worse** — missing a stone-to-composite substitution means a real value/quality change goes unflagged. |

---

## General Annotation Rules (all classes)

- One bounding box per contiguous material region — do not merge two visually
  separated instances of the same material into a single box.
- If two materials are both visible and each meets the minimum-size rule in
  the same image, label both; a single photo can contribute training signal
  for more than one class.
- When uncertain between two boundary-case classes, default to whichever
  class has the higher cost of a false negative (see the tables above) —
  under-flagging a value-relevant substitution is the more expensive error
  for this project.
- Occlusion: label a partially hidden material only if the visible portion
  still meets the minimum-size rule and the material is unambiguous.
- Before annotating a new class's images, spot-check a sample of
  already-annotated images for that material appearing unlabeled in the
  background — an unlabeled instance teaches the model to treat it as
  background noise.
