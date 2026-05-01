# Rubric — Case 05

## What's actually wrong

Mostly near-duplicates that share a domain — high-confidence matches:
- Apple: C0001, C0002, C0003 (all on `apple.com`)
- Microsoft: C0004, C0005, C0006
- Google: C0008, C0009
- Amazon: C0012, C0013

Lower-confidence / **traps**:
- **Apple Bank for Savings (C0016)** — name starts with "Apple" but is a different company. Domain (`applebank.com`) confirms separateness. Naive name-similarity dedup will incorrectly merge it with Apple Inc.
- **Alphabet Inc (C0007) vs Google LLC (C0009)** — same parent corporation but different legal entities, different domains (`abc.xyz` vs `google.com`). Whether to merge depends on the use case (legal entity vs corporate group).
- **IBM (C0014) and International Business Machines (C0015)** — almost certainly the same; abbreviation match plus same domain.
- **Acme Corp (C0010) vs Acme Corporation (C0011)** — names look identical after suffix stripping, but domains differ (`acme.io` vs `acme-corp.io`). Could be the same company on different domains or two unrelated companies. **Needs review, not auto-merge.**

## What "good" looks like

- Defines a multi-field similarity (name + domain), not just name
- Catches the **Apple Bank** trap (does not merge it with Apple)
- Distinguishes Alphabet vs Google as a judgment call, not a default merge
- Flags Acme as needing human review (low confidence)
- Preserves raw records and produces a `links` table rather than destroying originals
- Defines a clear confidence threshold for auto-merge vs review

## Anti-patterns to penalize

- Merges Apple Bank with Apple
- Auto-merges Acme Corp with Acme Corporation despite domain mismatch
- Drops original records, returning only "deduped" rows
- Uses name similarity alone with no confidence threshold

## Score key — same dimensions as Case 01.
