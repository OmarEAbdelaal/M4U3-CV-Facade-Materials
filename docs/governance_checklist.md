# Governance Checklist — Facade Material Detection

One-page record of privacy, data, risk and licensing decisions for this
project. Status: ✅ done · ⚠️ partly done / known gap · ⬜ to do.

---

## 1. Privacy and consent

| Check | Status | Note |
|---|---|---|
| Images of private individuals or client projects excluded | ✅ | All images are public-domain or openly licensed photos from Wikimedia Commons. No client, employer or site photos are used. |
| People and vehicle licence plates in images | ⚠️ | Facade photos taken from the street can include passers-by and parked cars (the baseline notebook detected a car and a bicycle in two images). **Faces and plates were not blurred in dataset version 4.** |
| Blur incidental faces and plates | ⬜ | Blur (or crop out) people and plates before the next dataset version. Identity is never needed to answer "which material is this facade?". |
| Consent | ✅ | Not required for the source material: images were already published under open licenses by their authors. No new photos of people were taken. |

## 2. Data minimisation

- The task only needs the **visible facade surface**. People, vehicles,
  street furniture and interiors carry no information the model needs.
- Only the image pixels and bounding-box labels are stored. No location
  metadata, building addresses or owner information is collected. EXIF
  orientation data is stripped by Roboflow's Auto-Orient step.
- Images are resized to 640 × 640 px, which is enough for material
  recognition and reduces detail on any incidental people.
- **Next version:** crop training images to the facade where people or
  vehicles dominate the frame (material crops only).

## 3. Limitations — when not to use this model

> **This model is a screening aid, not a certified compliance
> verification.** It can flag that a facade *appears* to show a material
> different from the one specified, so that a person checks it. It cannot
> confirm that a building complies with a specification, a contract or a
> regulation.

Do **not** use it to:

- certify that the installed material matches the specification, or approve
  a payment or handover on its output alone;
- make any fire-safety or life-safety judgement (for example, whether a
  cladding panel is combustible). The model sees appearance only; it cannot
  see core material, fire rating, fixings or thickness;
- judge materials that look alike by design: stone-look composite panels,
  brick slips, glass-look reflective panels, painted concrete;
- analyse images that are night-time, heavily shadowed, black-and-white,
  very distant, or taken at steep angles; these are under-represented in
  the training data;
- identify or monitor people.

Every flag must be reviewed by a qualified person (architect, facade
consultant or site engineer) before any decision.

## 4. Risk note — false negatives vs. false positives

| Error | What happens | Consequence in this use case | Priority |
|---|---|---|---|
| **False negative** (a material is missed) | A substitution goes unflagged, e.g. composite panels installed instead of the specified natural stone | A value-engineering change that affects cost, quality or durability is never reviewed | **Higher cost: minimise first** |
| **False positive** (a material is reported that isn't there) | A reviewer checks a facade that is actually correct | Wasted review time; loss of trust if frequent | Lower cost, but must stay manageable |

Operating choice: favour **recall** over precision for `glass`,
`cladding_panel`, `stone_cladding` and `exposed_concrete`, the classes where
substitutions are most common, and accept extra review work from false
alarms. A lower confidence threshold is the lever for this.

## 5. Licensing

| Item | License | Note |
|---|---|---|
| Code and notebooks in this repository | **MIT** | See [`LICENSE`](../LICENSE). |
| Documentation (`docs/`, README) | MIT | Same file. |
| Dataset images | **Each image keeps its own source license** (CC0, public domain, CC BY or CC BY-SA, from Wikimedia Commons) | Author, license and source page for every released image: [`data/attribution.csv`](../data/attribution.csv). |
| Dataset as a whole (GitHub Releases `v1.0` and `v1.1`, Roboflow Universe) | ✅ **CC BY-SA 4.0** | Checked 27 Sep: 184 of 253 images are CC BY-SA, so ShareAlike applies to the whole dataset. CC0, public-domain and CC BY images are compatible. Stated in the README, `data/README.md` and the Release notes. ⬜ Change the license field on Roboflow Universe from CC BY 4.0 to CC BY-SA 4.0 (Project → Settings). |
| New-image test photos | Own license each (CC BY-SA or CC BY) | Used only for inference; listed in [`data/new_images.csv`](../data/new_images.csv). |
| Ultralytics library (YOLOv8) | AGPL-3.0 | Used as a dependency, not redistributed. |
| Trained weights (`best.pt`) | AGPL-3.0 | Ultralytics treats models trained with its library as covered by AGPL-3.0, so the weights are shared under AGPL-3.0, not MIT. Commercial use would need an Ultralytics Enterprise license. |

## 6. Data provenance and human oversight

| Check | Status | Note |
|---|---|---|
| Source of every image recorded | ✅ | Collection notebooks logged URL, license and author for every image; the lists are committed in [`data/`](../data/) (`attribution.csv`, `sources_*.csv`). |
| Dataset frozen and verifiable | ✅ | Release `v1.0` (Roboflow export), SHA256 `134e41be…51cd`; Release `v1.1` (cleaned by notebook 01b, reproducible byte for byte), SHA256 `bdcf62ff…4187164`. |
| No API keys or secrets in the repository | ✅ | Notebooks download the dataset keylessly; no key appears in any cell. |
| Human in the loop | ✅ by design | Output is a list of flags for review; nothing is approved or rejected automatically. |
| Known labelling inconsistencies documented | ✅ | See `project_log.md` entry 8, `error_analysis.md` and `sam_exploration_notes.md`. |
| Model performance stated honestly | ✅ | Run 04 misses the recall target (0.35 vs 0.50); README section 6 and the report say so, and the model is described as a screening aid only. |
