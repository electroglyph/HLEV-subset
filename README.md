# HLE-Verified — Gold 200 (text-only)

A random 200-item, **image-free** subset of the **Gold** split of
[HLE-Verified](https://github.com/SKYLENAGE-AI/HLE-Verified).

The parent benchmark contains 2,500 audited items split into Gold (668),
Revision (1,143), and Uncertain (689). Of the 668 Gold items, 504 contain no
image data at all. This folder contains a reproducible simple random sample of
200 items from those 504, so every item is answerable from text alone.

## Contents

| File | Purpose |
|------|---------|
| `gold_200.jsonl` | The 200 selected items, one complete source record per line. |
| `manifest.json` | Source commit/checksum, sampling frame, seed, selected line indices, item IDs, and composition stats. |
| `README.md` | This file. |

The dataset is self-contained: there are no image files, no base64 blobs, and
no external assets to download.

## Provenance and selection

- Source repository: `SKYLENAGE-AI/HLE-Verified`
- Source commit: `b705e0fb541c025a1532ce0d60d70ae2f53b00e0`
- Source file: `data/Gold_subset.jsonl` (668 items)
- Source SHA-256: `f03c2632400970ffe5167d5ecde169ad38d34ce4dceb2eb8cfb54ece9c5fae00`
- Sampling frame: the 504 Gold items with `image == ""`,
  `image_preview == null`, and `rationale_image == null`
- Selection method: uniform simple random sample **without replacement**
- Seed: `42`
- The 200 source line indices are sorted so the subset preserves the source
  file order.
- Output SHA-256: `89e824d1462536689ac591b23165397b8148e8dc5af009cc2c831f9a5be8f8c1`

Reproduce from a checkout of the parent repository:

```python
import json, random

with open("data/Gold_subset.jsonl") as f:
    lines = f.readlines()

rows = [json.loads(line) for line in lines]
eligible = [
    i for i, r in enumerate(rows)
    if not r["image"]
    and r["image_preview"] is None
    and r["rationale_image"] is None
]
assert len(eligible) == 504

indices = sorted(random.Random(42).sample(eligible, 200))
with open("gold_200.jsonl", "w") as f:
    f.writelines(lines[i] for i in indices)
```

The exact selected indices and item IDs are listed in `manifest.json`.

## Composition

| Category | Items |
|----------|------:|
| Math | 89 |
| Computer Science/AI | 36 |
| Humanities/Social Science | 21 |
| Physics | 19 |
| Biology/Medicine | 14 |
| Other | 13 |
| Engineering | 5 |
| Chemistry | 3 |
| **Total** | **200** |

| Answer type | Items |
|-------------|------:|
| `exactMatch` | 139 |
| `multipleChoice` | 61 |

No item has a question image, image preview, or rationale image.

## Record format

Each line is a complete record copied unchanged from the parent Gold split, so
the schema is identical to HLE-Verified's. The fields needed for evaluation
are:

- `id` — unique item identifier
- `question` — the problem statement
- `answer` — reference answer (a letter for multiple choice, free-form text
  for `exactMatch`)
- `answer_type` — `multipleChoice` or `exactMatch`
- `category`, `raw_subject` — topic labels for reporting breakdowns

Reference-solution and audit metadata are retained as well:

- `rationale` — reference rationale
- `verify_meta_info`, `Verified_Classes` — HLE-Verified audit metadata
  (`Verified_Classes` is `"Gold subset"` for every item)
- `canary` — benchmark contamination canary string

The three image fields are present for schema compatibility but empty:
`image` is `""`, and `image_preview` and `rationale_image` are `null` for all
200 records. There is no base64 or other binary data anywhere in the file.

## Loading

```python
import json

records = [json.loads(line) for line in open("gold_200.jsonl")]

for r in records:
    prompt = r["question"]
    answer = r["answer"]
    answer_type = r["answer_type"]
```

## Evaluation note

`multipleChoice` items can be scored by comparing the selected letter with
`answer`. `exactMatch` items contain free-form answers; the original HLE
protocol uses an LLM judge for these rather than naive string equality, so use
the same judging protocol you use for the parent benchmark when reporting
scores. Scores on this 200-item subset are not directly comparable to scores on
the full 668-item Gold split.
