# known_bad/ — quarantined files

**Nothing in this directory is part of the release data.** These files are kept
only so that the defect is auditable: they are reproduced exactly as the
annotator submitted them, and they are excluded from `annotations/`, from every
count in the top-level `README.md`, and from the label statistics in
`alias_map.csv`. They remain covered by `SHA256SUMS.txt`, and `manifest.csv`
keeps a row for each with `status = excluded` and `path` pointing here.

## `b24fe5969a0d37b2cbdd637ad59cc19d_subtask_Human2.json`

Wrong-episode contamination. The file is filed under LIBERO-10 episode
`b24fe5969a0d37b2cbdd637ad59cc19d` (task
`turn_on_the_stove_and_put_the_moka_pot_on_it`, 290 frames), but:

* `Task_Info.TaskID` reads `"wipe wine"` — a task that is not in LIBERO at all;
  it is the only file in the whole collection whose `TaskID` disagrees with its
  directory.
* Its 9 segments span frames 0–1299, against an episode of 290 frames — more
  than four times the episode length.
* Its label sequence (`Grasp, Lift, Grasp, Lift, Move, Fold, Lower, Lift,
  Place`) does not describe a stove-and-moka-pot episode, and the three labels
  `Lift`, `Lower` and `Fold` occur nowhere else in Human2's work.

The labels evidently belong to a different, much longer episode from another
corpus. The file must **not** be rescaled or clamped onto this episode — the
frame numbers are not a unit error, they refer to other footage. Human2's
absorb annotation for this episode is therefore missing, and the LIBERO-10
Human2 absorb cell covers 29 of 30 episodes.
