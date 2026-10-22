# HLE-Verified — Gold Multiple-Choice (no image data)

**154** multiple-choice items from the **Gold** split of
[HLE-Verified](https://github.com/SKYLENAGE-AI/HLE-Verified) whose records
contain no image data at all.

The parent benchmark contains 2,500 audited items split into Gold (668),
Revision (1,143), and Uncertain (689). The Gold split has 209 multiple-choice
items:

- **154** have no image data in any image field → included here
- **55** contain image data → excluded
  - 36 have a question image
  - 19 have only a rationale figure (no question image)

The 19 rationale-only items are answerable from their question text, but their
source records include an image. They are excluded here so every record is
entirely image-free without editing the source data. If the requirement were
only "the question has no image," the count would be 173.

No sampling was applied: this is the complete eligible set under the no-image
criterion.

## Contents

| File | Purpose |
|------|---------|
| `gold_mc_text.jsonl` | The 154 items, one complete source record per line. |
| `manifest.json` | Source commit/checksum, selection criteria, item IDs, and composition stats. |
| `README.md` | This file. |

The records are unmodified copies from the parent Gold split, so they remain
drop-in compatible with HLE-Verified's schema.

## Provenance and selection

- Source repository: `SKYLENAGE-AI/HLE-Verified`
- Source commit: `b705e0fb541c025a1532ce0d60d70ae2f53b00e0`
- Source file: `data/Gold_subset.jsonl` (668 items)
- Source SHA-256: `f03c2632400970ffe5167d5ecde169ad38d34ce4dceb2eb8cfb54ece9c5fae00`
- Selection criteria: `answer_type == "multipleChoice"` **and**
  `image == ""` **and** `image_preview == null` **and**
  `rationale_image == null`
- Method: exhaustive (no sampling), records kept in source line order
- Output SHA-256: `41af3631244e6525a5b2c1c434199774fa1ff77201eb76829923abdd67af6fa4`

Reproduce from a checkout of the parent repository:

```python
import json

with open("data/Gold_subset.jsonl") as f:
    lines = f.readlines()

rows = [json.loads(line) for line in lines]
indices = [
    i for i, r in enumerate(rows)
    if r["answer_type"] == "multipleChoice"
    and not r["image"]
    and r["image_preview"] is None
    and r["rationale_image"] is None
]
assert len(indices) == 154

with open("gold_mc_text.jsonl", "w") as f:
    f.writelines(lines[i] for i in indices)
```

The exact selected indices and item IDs are listed in `manifest.json`.

## Composition

| Category | Items |
|----------|------:|
| Biology/Medicine | 37 |
| Math | 30 |
| Humanities/Social Science | 29 |
| Computer Science/AI | 25 |
| Other | 17 |
| Physics | 10 |
| Chemistry | 3 |
| Engineering | 3 |
| **Total** | **154** |

| Answer | Items |
|--------|------:|
| A | 21 |
| B | 28 |
| C | 28 |
| D | 24 |
| E | 23 |
| F | 10 |
| G | 7 |
| H | 3 |
| I | 4 |
| J | 2 |
| K | 2 |
| M | 1 |
| R | 1 |

All 154 items are `multipleChoice`. No record contains a question image,
image preview, or rationale image.

## Record format

Each line is a complete record copied unchanged from the parent Gold split.
The fields needed for evaluation are:

- `id` — unique item identifier
- `question` — the problem statement, including the answer options
- `answer` — the correct option letter
- `answer_type` — always `multipleChoice` here
- `category`, `raw_subject` — topic labels for reporting breakdowns

Reference-solution and audit metadata are retained:

- `rationale` — reference rationale text
- `original_rationale` — present on 15 items, where the source provides one
- `verify_meta_info`, `Verified_Classes` — HLE-Verified audit metadata
  (`Verified_Classes` is `"Gold subset"` for every item)
- `canary` — benchmark contamination canary string

The three image fields are empty in every record: `image` is `""`,
`image_preview` is `null`, and `rationale_image` is `null`. There is no base64
or other binary data anywhere in the file.

## Loading

```python
import json

records = [json.loads(line) for line in open("gold_mc_text.jsonl")]

for r in records:
    prompt = r["question"]
    correct_letter = r["answer"]
```

## Verification

Checked against the source Gold split:

- 154 output lines, 154 unique IDs, all `Verified_Classes == "Gold subset"`
- all 154 have `answer_type == "multipleChoice"`
- every `image` is `""`, every `image_preview` and `rationale_image` is `null`
- zero occurrences of `base64` in the output
- output SHA-256 matches the value above
- the reproduction snippet above produces a byte-identical file
- source split: 668 Gold items, 209 multiple-choice, of which 154 have no
  image data

## Evaluation note

Because every item is multiple choice, scoring is a direct comparison of the
model's selected letter against `answer` — no LLM judge is required. Scores
here are not directly comparable to scores on the full Gold split, which also
contains free-form `exactMatch` items and image-based questions.
