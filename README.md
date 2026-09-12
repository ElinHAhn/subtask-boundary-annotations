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

**absorb = 60 episodes × 3 annotators = 180 files.
fine = 30 LIBERO-Object episodes × 3 annotators = 90 files.
270 annotation files in total**, with no missing annotator/convention/episode
cell.

| Episode set | Episodes | absorb | fine | Annotators | Files |
|---|---|---|---|---|---|
| LIBERO-Object | 30 | yes | yes | Human1, Human2, Human3 | 30 × 3 × 2 = 180 |
| LIBERO-10 | 30 | yes | — (convention not defined) | Human1, Human2, Human3 | 30 × 3 = 90 |
| **Total** | **60** | **180** | **90** | **3** | **270** |

The fine convention is defined for LIBERO-Object only; LIBERO-10 episodes are
multi-object and multi-stage, and the fine decomposition was not specified for
them, so there are deliberately no `_S_` files under `libero_10/`.

Inside the data files and manifests the annotators are identified only as
**Human1**, **Human2** and **Human3**. The codes exist so that scoring stays
blind to who produced which labels, not to withhold credit: the annotators are
named in *Credits* below and in the dataset citation, and the mapping between
code and person is deliberately not published.

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
annotations/            270 files — the release data
  libero_object/<task_name>/<episode_hash>/
        <episode_hash>_subtask_Human1.json        absorb
        <episode_hash>_subtask_Human2.json        absorb
        <episode_hash>_subtask_Human3.json        absorb
        <episode_hash>_subtask_S_Human1.json      fine
        <episode_hash>_subtask_S_Human2.json      fine
        <episode_hash>_subtask_S_Human3.json      fine
  libero_10/<task_name>/<episode_hash>/
        <episode_hash>_subtask_Human{1,2,3}.json  absorb only
videos/                 episode videos, 120 MP4 files (~358 MB)
  <suite>/<task_name>/<episode_hash>/
        <episode_hash>_observations_image_raw_dsnone_fps10.mp4
        <episode_hash>_observations_image2_raw_dsnone_fps10.mp4
manifest.csv            one row per annotation file (270)
alias_map.csv           raw label -> canonical label, per annotator/convention
SHA256SUMS.txt          checksums for annotations/, manifest.csv, alias_map.csv
docs/
  ontology_v1.4.ko.md   three-layer action ontology + boundary definitions (Korean)
```

`manifest.csv` columns:
`suite, task, episode, annotator, convention, status, path, n_frames, fps,
n_seg, pattern, source_archive, orig_filename, sha256`.

* `status` is `ok` for **every row** in the current release — nothing is
  withheld or quarantined. The column is kept so the schema stays stable: it
  exists to carry a value other than `ok` if a future revision ever has to
  withhold a file, so a consumer should still filter on `status == "ok"` rather
  than assume it.
* `path` is the file's location in this repository, so a row can always be
  resolved to the exact bytes that were checksummed.
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

Measured over all **270 files (1000 segments)**:

| Annotator | Segments | Distinct raw labels | Labels needing remapping | Segments remapped |
|---|---|---|---|---|
| Human1 | 372 | 6 | 0 | 0 |
| Human2 | 337 | 10 | 3 (`Release` in absorb — partly, `Reach`, `Put`) | 53 |
| Human3 | 291 | 9 | 5 (`Pick`, `Put`, `Turn`, `Slide`, `carry`) | 167 |
| **Total** | **1000** | — | — | **220** |

Human1 used the given list exactly. Human2 coined `Reach` (7) and wrote
`Release` (46 absorb / 26 fine) and `Put` (3); `Lift`, `Lower` and `Fold` appear
nowhere in the release. Human3 coined `Pick` (80 — the dominant label in that
set and the one with no canonical equivalent), `Turn` (3), `Slide` (2) and the
lowercase `carry` (2), and used `Put` (51 absorb / 29 fine) for two different
spans.

**Any cross-annotator comparison must normalise through `alias_map.csv`
first.** A raw-label confusion matrix across annotators is meaningless.

**`Put` maps differently in the two conventions, and a single global rule is
wrong.** In the absorb files `Put` is the whole carry-and-release phase and maps
to `Place`; in the fine files the carry is already its own `Move` segment, so
`Put` denotes only the gripper-opening span and maps to `Release`.

**Human2's absorb `Release` needs a rule finer than the convention, because it
denotes two different acts.** Of its 46 absorb occurrences:

* **43 are the terminal set-down of a carried object** — `EngName` reads
  "release the *object* into/over the basket", and the segment follows a
  transport (`Move` 37, `Place` 3, `Grasp` 2, `Reach` 1). Here `Release` stands
  for the whole `Place` phase and maps to **`Place`**.
* **3 are letting go of a control** — `EngName` reads "Release the stove knob",
  and each immediately follows a `Rotate` of that same knob, in the three
  `turn_on_the_stove_and_put_the_moka_pot_on_it` episodes
  (`1e3755806766f68bdefdd683923905ca`, `3157fa8215d35be37996a7b6505f1733`,
  `b24fe5969a0d37b2cbdd637ad59cc19d`). Nothing is placed anywhere: the knob
  stays where it is and the gripper withdraws. Mapping these to `Place` would
  assert a set-down that does not happen, so here `Release` is **already
  canonical and maps to itself**.

The two cases are separated by two agreeing signals — the preceding segment type
(`Rotate` vs a transport) and the object named in `EngName` (a knob vs a carried
item) — and the split is exact, with no occurrence ambiguous between them, which
is why both rows carry confidence *high*. In the fine files Human2's `Release`
is canonical throughout and maps to itself.

`alias_map.csv` therefore carries **two rows** for `(Release, Human2, absorb)`,
one per canonical target, with the `note` column stating what distinguishes them
and how many occurrences fall in each. It is keyed on
`(raw_label, annotator, convention)` and that key is no longer sufficient on its
own — do not collapse it to a `raw -> canonical` dictionary, and do not apply
either `Release` row without checking the segment's context.

Two mappings are uncertain and are marked as such in `alias_map.csv`: Human3's
`Put` in the fine convention (`Release`, confidence *medium*) and Human3's
`Turn` (`Rotate`, confidence *low* — it is the stove-knob motion and cannot be
settled from the label files alone; it needs the video).

---

## Frame indexing

* Frame indices are **0-based** and **inclusive on both ends**: a segment
  `[start, end]` contains frame `end`.
* `start` of segment *k+1* equals `end` of segment *k* plus 1; there are no gaps
  and no overlaps. This holds for all 270 files, and every file's first segment
  starts at frame 0.
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

These are shipped as the annotators produced them — nothing has been silently
repaired. All 270 files are internally contiguous (no gaps, no overlaps, first segment starts at 0), but the
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

**Last segment stops short of the final frame** — 3 files, all Human3 absorb:

| episode | suite / task | n_frames | last `EndFrameNum` | frames unlabelled |
|---|---|---|---|---|
| `ff3a33419101905ff33e961454b21b06` | libero_10 / put_the_white_mug_on_the_left_plate… | 292 | 276 | 15 |
| `b04096415d1942662fbb1098e72105ad` | libero_10 / put_the_white_mug_on_the_plate… | 249 | 231 | 17 |
| `398e4699f9e4519bc21544e7d50bab53` | libero_object / pick_up_the_alphabet_soup… | 152 | 149 | 2 |

**One annotation was resubmitted before release.** Human2's absorb file for
`b24fe5969a0d37b2cbdd637ad59cc19d` (libero_10 /
turn_on_the_stove_and_put_the_moka_pot_on_it) was initially submitted with
wrong-episode content — a different task's labels spanning 1299 frames against
an episode of 290 — and was replaced by a corrected annotation from the same
annotator. **The released file is the corrected one**: 5 segments,
`Reach | Rotate | Release | Grasp | Place`, ending exactly at frame 289. One
artefact of the original submission survives in it: `Task_Info.TaskID` still
reads `"wipe wine"`, so this remains the only file in the release whose `TaskID`
disagrees with its directory. The frame data and labels are correct; read the
task from the directory path or from `Task_Info.task_instruction` (which is
correct), not from `TaskID`.

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

If you use these annotations, cite the dataset:

```bibtex
@misc{l24annotations2026,
  title  = {LIBERO Subtask Boundary Annotations (L24):
            three annotators, two segmentation conventions},
  author = {An, Hyeyoung and Bae, Yoosung and Han, Hyonyoung},
  year   = {2026},
  url    = {https://github.com/ElinHAhn/subtask-boundary-annotations}
}
```

The annotations are the work of the three annotators named above. Inside the
data files they appear as `Human1`, `Human2` and `Human3` — the codes keep
scoring blind to who produced which labels, and the mapping from code to person
is deliberately not published, so that no individual's agreement score is
attributable. See *Credits* below.

The accompanying paper (in preparation) describes the ontology and the
measurement protocol; it will be listed here once published.

Please cite LIBERO as well; see [`ATTRIBUTION.md`](ATTRIBUTION.md).

---

## Credits

**Annotation:** Hyeyoung An, Yoosung Bae, Hyonyoung Han.

Each annotated the same 60 LIBERO episodes under the absorb convention and the
same 30 LIBERO-Object episodes under the fine convention — 90 episodes each,
independently, without seeing one another's work.
