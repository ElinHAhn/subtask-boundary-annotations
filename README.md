# LIBERO Subtask Boundary Annotations (L24)

Human subtask-boundary annotations for 60 episodes of the
[LIBERO](https://libero-project.github.io/) benchmark, labelled by **three
independent annotators** under **two explicit segmentation conventions**,
together with the three-layer action ontology used to score them.

The point of this release is not a larger label set. It is that the same
episodes are labelled by three people, and — on LIBERO-Object — twice under two
conventions that differ only in how the *approach* and *carry* phases are
treated, so that "annotators disagree" can be separated from "the convention
does not say where the boundary is."

---

## Licensing

| What | License |
|---|---|
| Scripts and code in this repository | MIT — see [`LICENSE`](LICENSE) |
| Annotations, manifests, ontology spec | CC BY 4.0 — see [`LICENSE-DATA`](LICENSE-DATA) |

This mirrors LIBERO's own split (code MIT, datasets CC BY 4.0). GitHub displays
only one license per repository, so the repository badge reads MIT; the data
license is the one that governs everything under `annotations/` and `docs/`.

Upstream attribution is in [`ATTRIBUTION.md`](ATTRIBUTION.md).

---

## Coverage

**absorb = 60 episodes × 3 annotators = 180 files, less 1 quarantined = 179.
fine = 30 LIBERO-Object episodes × 3 annotators = 90 files.
269 annotation files are shipped under `annotations/`** (270 were collected; one
was quarantined — see *Known defects*).

| Episode set | Episodes | absorb | fine | Annotators | Shipped files |
|---|---|---|---|---|---|
| LIBERO-Object | 30 | yes | yes | Human1, Human2, Human3 | 30 × 3 × 2 = 180 |
| LIBERO-10 | 30 | yes | — (convention not defined) | Human1, Human2, Human3 | 30 × 3 − 1 = 89 |
| **Total** | **60** | **179** | **90** | **3** | **269** |

**There is exactly one gap, and it is not an unlabelled episode.** The
LIBERO-10 **Human2 absorb** cell covers **29 of 30 episodes**: the file for
episode `b24fe5969a0d37b2cbdd637ad59cc19d` (task
`turn_on_the_stove_and_put_the_moka_pot_on_it`) is wrong-episode data and has
been quarantined to [`known_bad/`](known_bad/README.md). Every other
annotator × convention × episode cell is present. Annotators did submit all 270
files; 269 are usable.

| Cell | Expected | Present |
|---|---|---|
| LIBERO-10 absorb, Human2 | 30 | **29** |
| all other cells | — | complete |

The fine convention is defined for LIBERO-Object only; LIBERO-10 episodes are
multi-object and multi-stage, and the fine decomposition was not specified for
them, so there are deliberately no `_S_` files under `libero_10/`.

Annotators are identified only as **Human1**, **Human2**, **Human3**. No real
names appear anywhere in this repository.

Annotations on an internal real-robot corpus exist but are **not** released
here, because that corpus mixes several upstream datasets with incompatible
licenses.

The episode videos are included in this repository under `videos/`, laid out on
the same `<suite>/<task>/<episode>` paths as `annotations/` — two camera streams
per episode, 120 files, ~358 MB. Watching the video is the unambiguous way to
confirm which episode an annotation refers to — see *Episode identity* below.
There is no separate data deposit; this repository is the whole release.

---

## Contents

```
README.md
LICENSE                 MIT (code)
LICENSE-DATA            CC BY 4.0 (annotations, manifests, ontology)
ATTRIBUTION.md          upstream attribution (LIBERO, LeRobot conversion)
.gitignore
annotations/            269 files — the release data
  libero_object/<task_name>/<episode_hash>/
        <episode_hash>_subtask_Human1.json        absorb
        <episode_hash>_subtask_Human2.json        absorb
        <episode_hash>_subtask_Human3.json        absorb
        <episode_hash>_subtask_S_Human1.json      fine
        <episode_hash>_subtask_S_Human2.json      fine
        <episode_hash>_subtask_S_Human3.json      fine
  libero_10/<task_name>/<episode_hash>/
        <episode_hash>_subtask_Human{1,2,3}.json  absorb only
                                                  (one Human2 file quarantined)
videos/                 episode videos, 120 MP4 files (~358 MB)
  <suite>/<task_name>/<episode_hash>/
        <episode_hash>_observations_image_raw_dsnone_fps10.mp4
        <episode_hash>_observations_image2_raw_dsnone_fps10.mp4
known_bad/              NOT release data — quarantined, kept for audit only
  README.md             what is wrong with each quarantined file
  b24fe5969a0d37b2cbdd637ad59cc19d_subtask_Human2.json
manifest.csv            one row per collected file (270), shipped and quarantined
alias_map.csv           raw label -> canonical label, per annotator/convention
                        (counted over the 269 shipped files only)
SHA256SUMS.txt          checksums for annotations/, known_bad/, manifest.csv, alias_map.csv
docs/
  ontology_v1.4.ko.md   three-layer action ontology + boundary definitions (Korean)
```

`manifest.csv` columns:
`suite, task, episode, annotator, convention, status, path, n_frames, fps,
n_seg, pattern, source_archive, orig_filename, sha256`.

* `status` is `ok` for the 269 shipped files and `excluded` for the 1 quarantined
  file. **Filter on `status == "ok"` before computing anything.**
* `path` is the file's location in this repository, so an `excluded` row points
  into `known_bad/` rather than `annotations/`.
* `pattern` is the segment labels joined by `|` in order, **raw** — not
  normalised through `alias_map.csv`.

Verify the release with:

```
sha256sum -c SHA256SUMS.txt
```

---

## The two conventions

A single pick-and-place motion can be cut in two defensible ways.

**absorb** — what annotators do by default. The reach toward the object is part
of `Grasp`; the carry toward the goal is part of `Place`.

```
Grasp ................ | Place ................ | Move
```

**fine** — the transport phases are their own segments.

```
Move | Grasp | Move | Place | Move
```

Both label sets describe the *same* episodes and the same underlying motion.
Files ending `_subtask_Human{N}.json` are absorb; files ending
`_subtask_S_Human{N}.json` are fine.

Why both: the two conventions do not merely move a boundary, they change which
instants count as boundaries at all. On the 30 LIBERO-Object episodes the
Human1–Human2 pair agrees on segment *labels* considerably better under the fine
convention, because each fine segment has one physical cause rather than a
bundled phase; boundary localisation goes the other way, with absorb scoring the
higher boundary F1 simply because it asks for about half as many boundaries. The
conventions are a trade-off, not a ranking, and the release contains both so
that either can be measured.

A caution for anyone comparing boundary types by name across the two files: the
same type name can denote different physical events, so a raw absorb-vs-fine
score is not a like-for-like comparison. In the absorb files `ApproachOnset` is
the `Place -> Grasp` transition, and in 38 of 46 cases it falls on the same
frame as the preceding `Place` end, so it is not an independent instant. In the
fine files the corresponding boundary is the gripper-closing onset,
`Move -> Grasp`. Match by event, not by name.

### Where a boundary sits

**Every boundary is placed at the onset of the transition** — the first frame at
which the change is observable. Completion (`...Done`, e.g. the frame at which
the gripper finishes opening) is recorded as an attribute, never as a boundary.
This is the one rule that, once applied consistently, reconciles the label sets.

---

## Vocabulary

Annotators were given a fixed label list (the canonical set: `Move`, `Grasp`,
`Place`, `Idle`, `Other`, `Open`, `Close`, `Push`, `Pull`, `Rotate`, `Insert`,
`Align`, `Hold`, `Lift`, `Lower`, `Reach`, `Release`, `Put`, `Pour`, `Wipe`,
`Fold`). **Two of the three invented their own verbs anyway, and one reused a
listed verb for a span the list does not mean.** The released JSON keeps each
annotator's own terms verbatim; [`alias_map.csv`](alias_map.csv) records, for
every distinct raw label per annotator per convention, its count, its canonical
mapping, and a confidence.

Measured over the **269 shipped files (995 segments)**; the quarantined file's
9 segments are excluded and accounted for separately below:

| Annotator | Segments | Distinct raw labels | Labels needing remapping | Segments remapped |
|---|---|---|---|---|
| Human1 | 372 | 6 | 0 | 0 |
| Human2 | 332 | 10 | 3 (`Release` in absorb, `Reach`, `Put`) | 54 |
| Human3 | 291 | 9 | 5 (`Pick`, `Put`, `Turn`, `Slide`, `carry`) | 167 |
| **Total** | **995** | — | — | **221** |

Reconciliation: 995 shipped + 9 in the quarantined file = 1004 segments across
all 270 collected files. The quarantined file contributes `Grasp` 2, `Lift` 3,
`Move` 1, `Fold` 1, `Lower` 1, `Place` 1 — all to Human2 / absorb, which is why
Human2's totals are 332 rather than 341.

**`Lift`, `Lower` and `Fold` appear nowhere in the shipped data.** All five of
those segments came from the quarantined file alone, so Human2's distinct-label
count is 10 over the release, not the 13 seen across the raw submission. Any
earlier tally that lists Human2 as using `Lift` (3), `Lower` (1) or `Fold` (1)
is counting contaminated data.

Human1 used the given list exactly. Human2 coined `Reach` (6) and wrote
`Release` (45 absorb / 26 fine) and `Put` (3). Human3 coined `Pick` (80 — the
dominant label in that set and the one with no canonical equivalent), `Turn`
(3), `Slide` (2) and the lowercase `carry` (2), and used `Put` (51 absorb / 29
fine) for two different spans.

**Any cross-annotator comparison must normalise through `alias_map.csv`
first.** A raw-label confusion matrix across annotators is meaningless.

**`Put` maps differently in the two conventions, and a single global rule is
wrong.** In the absorb files `Put` is the whole carry-and-release phase and maps
to `Place`; in the fine files the carry is already its own `Move` segment, so
`Put` denotes only the gripper-opening span and maps to `Release`. The same
applies to Human2's `Release`: in the absorb files it stands for the full
`Place` phase, while in the fine files `Release` is already canonical and maps
to itself. `alias_map.csv` is keyed on `(raw_label, annotator, convention)` for
exactly this reason — do not collapse it to a `raw -> canonical` dictionary.

Two mappings are uncertain and are marked as such in `alias_map.csv`: Human3's
`Put` in the fine convention (`Release`, confidence *medium*) and Human3's
`Turn` (`Rotate`, confidence *low* — it is the stove-knob motion and cannot be
settled from the label files alone; it needs the video).

---

## Frame indexing

* Frame indices are **0-based** and **inclusive on both ends**: a segment
  `[start, end]` contains frame `end`.
* `start` of segment *k+1* equals `end` of segment *k* plus 1; there are no gaps
  and no overlaps. This holds for all 269 shipped files (and for the quarantined
  one), and every file's first segment starts at frame 0.
* Indices are frame numbers, not timestamps. Convert with the episode's own
  `fps` from `manifest.csv` — do **not** assume a single fps across the release.
  Tolerances stated in seconds (e.g. τ = 0.5 s) are converted per episode.

## Episode identity

`episode_index` is the index in the **LeRobot conversion**
(`HuggingFaceVLA/libero`), not LIBERO's original demonstration number, and not
an index within a suite. Always read it together with the `suite` column of
`manifest.csv`. If you need certainty, match against `videos/`: an episode's
footage sits at the same `<suite>/<task>/<episode_hash>` path as its
annotations, so the mapping is exact and needs no index arithmetic.

---

## Known defects

Apart from the one quarantined file, these are shipped as the annotators
produced them — nothing has been silently repaired. All 269 shipped files are
internally contiguous (no gaps, no overlaps, first segment starts at 0), but the
following do not end where the episode ends. They are **real annotations with a
boundary defect, not wrong data**, so they stay in `annotations/`.

**How a scorer should treat them:** clamp the final `EndFrameNum` to
`n_frames - 1` for the 5 small fine-convention overruns, treat the short tails
as unlabelled frames (do not extend the last segment to cover them), and **do
not rescale anything** — the overruns are a few frames of over-reach, not a unit
or frame-rate error.

**Last segment runs past the final frame** (`EndFrameNum > n_frames - 1`) — 5
shipped files, all Human2 fine:

| episode | suite / task | convention | n_frames | last `EndFrameNum` | overrun |
|---|---|---|---|---|---|
| `a15974726bfd5eeb046b1f1f16ee56c7` | libero_object / pick_up_the_chocolate_pudding… | fine | 148 | 158 | 11 |
| `05f3652bbb9cf09dc8e88732f0e36775` | libero_object / pick_up_the_butter… | fine | 149 | 153 | 5 |
| `058b705d3289ac54f0398a45eae5e3eb` | libero_object / pick_up_the_cream_cheese… | fine | 133 | 137 | 5 |
| `d676c9a46165d27cc3bd1cb50fdb65e7` | libero_object / pick_up_the_butter… | fine | 151 | 153 | 3 |
| `4267eb3d3d5cda5caeaa71480018ed63` | libero_object / pick_up_the_orange_juice… | fine | 129 | 130 | 2 |

A sixth overrun — `b24fe5969a0d37b2cbdd637ad59cc19d`, Human2 absorb, `n_frames`
290 against a last `EndFrameNum` of 1299 — is **not** in this table because it
is not a boundary defect: it is wrong-episode data and has been quarantined
(see below).

**Last segment stops short of the final frame** — 3 files, all Human3 absorb:

| episode | suite / task | n_frames | last `EndFrameNum` | frames unlabelled |
|---|---|---|---|---|
| `ff3a33419101905ff33e961454b21b06` | libero_10 / put_the_white_mug_on_the_left_plate… | 292 | 276 | 15 |
| `b04096415d1942662fbb1098e72105ad` | libero_10 / put_the_white_mug_on_the_plate… | 249 | 231 | 17 |
| `398e4699f9e4519bc21544e7d50bab53` | libero_object / pick_up_the_alphabet_soup… | 152 | 149 | 2 |

**Quarantined: wrong-episode contamination (1 file).**
`b24fe5969a0d37b2cbdd637ad59cc19d_subtask_Human2.json` carries
`Task_Info.TaskID = "wipe wine"` and 9 segments spanning frames 0–1299, against
an episode of 290 frames. It is the only file in the collection whose `TaskID`
does not match its directory, and its labels (`Grasp, Lift, Grasp, Lift, Move,
Fold, Lower, Lift, Place`) do not describe a stove-and-moka-pot episode. It has
been moved out of `annotations/` into [`known_bad/`](known_bad/README.md); it is
**not release data** and is kept only so the defect is auditable. It must not be
rescaled or clamped — the frame numbers refer to other footage. Consequence:
Human2 has no usable absorb annotation for this episode, which is the single
coverage gap noted above.

**Genuine absorb/fine disagreement**, `76ac76f286917e62fbd35d9560e5587b`
(libero_object / pick_up_the_bbq_sauce…, Human3): the absorb file ends its
`Pick` at frame 68 while the fine file places its boundaries at 50 / 57 / 116,
an 11-frame difference at the grasp-completion boundary. The fine file is
therefore **not** a strict refinement of the absorb file for this episode. The
seven other absorb/fine boundary mismatches in Human3's LIBERO-Object set are
off-by-one and are re-annotation jitter, not a different decomposition.

**Metadata heterogeneity (not a label defect).** `Task_Info` key sets differ by
source package: Human2's files and Human1's LIBERO-Object files carry
`Convention` and `Annotator` keys (a few Human1 files also carry
`ConventionSpec`), while Human3's files and Human1's LIBERO-10 files do not.
The reason is that Human1's absorb labels existed in two places, and the two
copies were deliberately not merged: Human1's **LIBERO-Object** absorb files are
taken from his packaged fine-convention archive, which adds the `Convention` and
`Annotator` provenance keys and keeps each absorb file beside its fine sibling;
Human1's **LIBERO-10** absorb files are taken from the raw annotation folder,
which has no such keys. The two copies of the 30 LIBERO-Object absorb files were
compared: their `SubTasks` arrays and all shared `Task_Info` values are
identical, and the packaged copies differ *only* by those extra keys, so nothing
was lost by preferring them. Every JSON is copied byte-for-byte from its source,
so no keys were added or removed to harmonise them. Read `annotator` and
`convention` from `manifest.csv` or from the filename, never from `Task_Info`.

---

## Citation

```bibtex
@misc{hahn2026l24,
  title  = {A Three-Layer Action Ontology and Measurement Protocol
            for Robot Subtask Boundaries},
  author = {Hahn, Elin and others},
  year   = {2026},
  note   = {Annotation data: https://github.com/ElinHAhn/subtask-boundary-annotations}
}
```

Please cite LIBERO as well; see [`ATTRIBUTION.md`](ATTRIBUTION.md).
