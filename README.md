# Headstone

**The public checkpoint record for [Gardener](https://github.com/deiotte/gardener).**

Gardener is a project for replacing legacy C functions with Rust while keeping evidence of what was checked. It keeps a local ledger of its work: an ordered record linked together with cryptographic hashes.

Headstone publishes small checkpoints from that ledger so later changes to already anchored history can be detected. The ledger and its supporting records stay with Gardener; this repository holds the checkpoints.

If you arrived here and found a file full of numbers and hashes, that is the purpose of the repository.

## A concrete example

Suppose Gardener has recorded a batch of C function behavior and prepared translation tasks. It adds references to those records to its local ledger, then publishes a checkpoint here.

Later, someone changes an earlier ledger entry and recalculates the local chain. The checkpoint in Headstone still contains the previously published hash. Checking the altered ledger against that checkpoint exposes the mismatch.

Deleting enough ledger entries to remove an anchored position is detected too.

This makes changes to anchored history detectable. It does not establish that the original work was correct.

## What is in this repository?

[**`anchors.jsonl`**](anchors.jsonl) is the machine-readable checkpoint record. JSON Lines means each line is a separate JSON object.

The first line identifies the format:

```json
{"kind":"anchors","version":"1"}
```

Each subsequent line contains an anchor. This checkpoint marks Gardener's first full pass, at ledger position 67:

```json
{"seq":"67","head":"bd2c4ee3d2002d3b8cfe296b8e4539223332efb884bf3ff7b3bfaf67529590de"}
```

| Field | Meaning |
|---|---|
| `seq` | The local ledger line's position, counted from zero. Position 67 is the 68th entry. |
| `head` | SHA-256 of that ledger line's exact bytes, including its ending newline. |

Both values are stored as strings. Anchor positions strictly increase; gaps are normal because each anchor records a checkpoint rather than every individual ledger entry.

Each ledger line includes the preceding line's hash, so a checkpoint commits to the chain leading up to that position. Ledger entries also reference supporting records by their content hashes.

## How Gardener uses it

For its anchored recording workflow, Gardener:

1. Reads Headstone over public HTTPS and checks its local ledger against every published anchor before starting work.
2. Refuses to proceed if Headstone cannot be read or the ledger fails those checks.
3. After recording new work, appends a checkpoint and pushes an anchor commit using a dedicated SSH deploy key.
4. Reads the result back over public HTTPS and confirms both the commit and the file's exact contents.

A push without successful read-back confirmation is reported as an unsuccessful anchoring operation. Re-anchoring the same latest ledger entry adds no duplicate checkpoint.

The [commit history](https://github.com/deiotte/headstone/commits/main/) shows the sequence of publications.

## What an anchor establishes

An anchor provides a public reference against which the corresponding local ledger can be checked. Its value depends on preserving the published anchor history and retaining the ledger and supporting records.

It does **not** establish:

- That a Rust replacement is correct, safe, or equivalent for every possible input.
- That the recorded tests were sufficient or their results truthful.
- That work was unchanged before its first anchor.
- An independent certification, trusted timestamp, or guarantee against the repository administrator.

Headstone cannot reconstruct a lost ledger or its supporting records from hashes. Public readers can inspect the anchors and their history; checking the underlying work also requires access to the corresponding Gardener evidence.

## Where to start

- **Browse the checkpoints:** open [`anchors.jsonl`](anchors.jsonl).
- **See when checkpoints were published:** inspect the [commit history](https://github.com/deiotte/headstone/commits/main/).
- **Explore the implementation:** visit [Gardener](https://github.com/deiotte/gardener). That repository is currently private, so access is required.

There is no application to install or service to run here. This repository's job is to preserve Gardener's public ledger checkpoints.
