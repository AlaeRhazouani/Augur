# Decisions

Each entry: what I decided, and why.

## 1. Exclude MAL entries from Track A labels
**Date:** 2026-09-26
**Decision:** Skip every advisory whose id starts with `MAL-`.
**Why:** MAL entries are packages built as malware, not bugs in real packages.
Track A predicts bugs in popular real packages, so they don't belong in its labels.
They are also most of the raw data (221,943 of 229,410 npm files, 11,774 of 25,726 PyPI files),
so keeping them would completely distort the counts.
Some MAL entries may describe a hijacked real package; revisit them for Track B.

## 2. Skip withdrawn advisories
**Date:** 2026-09-26
**Decision:** Skip every advisory that has a `withdrawn` date.
**Why:** A withdrawn advisory was retracted (a false alarm, a duplicate, or a disputed report).
It is not a real event, so it must not create a positive label.

## 3. Skip affected entries that have no package
**Date:** 2026-09-26
**Decision:** Inside an advisory, skip any `affected` entry without a `package` field.
**Why:** Without a package name we cannot tell which package the bug belongs to, so it can't be labeled.
Only one entry in the whole dataset is like this: `EEF-CVE-2026-56812`, which is about the
Phoenix framework (Elixir) and only lists a git repository.

## 4. Ignore GIT ranges
**Date:** 2026-09-26
**Decision:** When collecting fixed versions, skip ranges whose type is `GIT`.
**Why:** In a GIT range, the "fixed" values are commit hashes, not version numbers.
We need version numbers to look up release dates on PyPI and npm.

## 5. Merge rows with the same advisory and package
**Date:** 2026-09-26
**Decision:** Group rows by (`id`, `package`) and keep all their fixed versions in one list.
**Why:** An advisory lists a package once per version line it fixes
(for example saleor 3.9, 3.10 and 3.11 in GHSA-r8qr-wwg3-2r85), which created several rows for one bug.
Every fixed version is kept, because the earliest one to be released is the first moment the bug became visible.
Result: PyPI 18,448 rows to 14,467; npm 9,675 to 7,719.

## 6. Merge the same bug across databases, keeping the earliest date
**Date:** 2026-09-26
**Decision:** Give every row a bug key (its own GHSA id, or the GHSA id found in its aliases,
otherwise its own id), group by (`package`, bug key), and keep the earliest published date.
**Why:** PyPI has two databases (GHSA and PYSEC) describing the same bugs, while npm only has GHSA.
Without merging, PyPI packages would have their advisories counted twice and look riskier than npm packages.
The earliest date is kept because the label is anchored on the first moment the bug became known.
Example: trac, CVE-2005-4644, is dated 2022-05-01 by GHSA but 2005-12-31 by PYSEC.
Result: PyPI 14,467 rows to 7,778; npm 7,719 to 7,718.
