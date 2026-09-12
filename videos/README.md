# videos/

The episode videos the annotations refer to, laid out exactly like
`annotations/` so that labels and footage sit at the same path:

```
videos/<suite>/<task_name>/<episode_hash>/
      <episode_hash>_observations_image_raw_dsnone_fps10.mp4
      <episode_hash>_observations_image2_raw_dsnone_fps10.mp4
```

Two camera streams per episode — the `image` and `image2` streams of the source
LeRobot conversion — rendered at 10 fps with no temporal downsampling
(`raw`, `dsnone`), as the file names record. 60 episodes x 2 streams = **120
files, ~358 MB**.

The point of shipping them is episode identity: the directory name *is* the
episode hash used throughout `manifest.csv` and the annotation paths, so an
episode can be identified by watching it, without resolving `episode_index`
against any external dataset revision.

Frame numbering in the annotations matches these videos frame for frame: 0-based,
inclusive on both ends. The videos are 10 fps; use the per-episode `fps` column
of `manifest.csv` rather than assuming it.

Provenance and licensing: the underlying demonstrations are from the LIBERO
benchmark (CC BY 4.0), redistributed via the HuggingFace LeRobot conversion.
See `../ATTRIBUTION.md`. The annotations are this project's work and are
licensed CC BY 4.0 as described in `../LICENSE-DATA`.
