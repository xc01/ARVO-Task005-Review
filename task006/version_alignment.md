# Task 006 Version Alignment

## Q1: Does image source match the official commit parent?

No. The vulnerability image source differs significantly from the official fix-commit parent (`10737165…`) source saved in Task 004.

| source | SHA-256 |
|---|---|
| image vul `OpenDDLParser.cpp` | `f52809b502c84231cae4a51121f8f35cf6925cfce1f4a806298591760b1f87d8` |
| Task 004 parent (official) | `b983346ebd95e387706d234b3db0573be994911969c618c20b721dd4287a9746` |
| image fix `OpenDDLParser.cpp` | `7429fc762dd71f60f7618299e5a642189a60e75f93d34a760c40824f8f173a7b` |
| Task 004 after (official fix commit) | `7429fc762dd71f60f7618299e5a642189a60e75f93d34a760c40824f8f173a7b` |

The fix image matches the official fix commit exactly. The vul image does not match the official parent.

The vul image source corresponds to upstream Assimp commit `575ce6136d4941a45c6ba894947d0e79fe032aff` (merge PR #3347, 2020-07-25), confirmed by Git metadata inside `/src/assimp/.git` and by `git show HEAD:path` hashing to the same file hash.

## Why the gap exists (confirmed, not inferred)

`575ce613` is a direct **ancestor** of the official parent `10737165` (`git merge-base --is-ancestor` returns true). The build tree is checked out at a revision roughly 15 months older than the revision ARVO records as the fix parent.

Five upstream commits touching `OpenDDLParser.cpp` sit between the two revisions:

| commit | date | subject |
|---|---|---|
| `57e691e28` | 2020-10-25 | openddl: latest greatest. |
| `7e1a8f09c` | 2020-10-26 | more openddl-fixes |
| `e1052f421` | 2021-01-06 | Update OpenDDLParser.cpp |
| `558457e5b` | 2021-05-04 | [openddlparser] Remove default log handler and unsolicited output. |
| `16c3f8222` | 2021-07-26 | closes issue #3975: Use latest version of OpenDDL-Parser |

`16c3f8222` is the decisive one: it rewrote `OpenDDLParserUtils.h` from 260 to 502 lines and introduced `chartype_table`, which the official patch then hardens. That table does not exist in the vul image source at all (`isNumeric` is a simple `'0' <= in <= '9'` range compare).

Consequence: **the official fix diff does not apply to the vul image source.** Verified with `git apply --check`; it fails at the first hunk (`OpenDDLParser.cpp:292`) because the surrounding code was rewritten by those intermediate commits.

## Q2: Can ASan line 551 be matched in actual source?

Yes. In the vulnerability image source (`575ce613`), line 551 is exactly:

```c
551    while( !isSeparator( *in ) && 
552            !isNewLine( *in ) && ( in != end ) && 
```

This is the dangerous loop: it dereferences `*in` before checking `in != end`. The address `0x1230f00` from the ASan report resolves via `addr2line` to `OpenDDLParser.cpp:551`, confirming binary-to-source alignment.

In the Task 004 official parent source, the same loop is at lines 551–552, with the same dereference-before-bound-check pattern. The line number coincidence is accidental: the two files are 1031 lines each but differ in 1024 changed lines (whitespace, formatting, `nullptr` vs `ddl_nullptr`, `chartype_table` introduction).

In the fix image source, the loop was reordered:

```c
556    while ((in != end) && !isSeparator(*in) && !isNewLine(*in) &&
```

The bound check was moved to the front, fixing the OOB read.

## Q3: Which source version did the binary use?

Confirmed. DWARF compilation unit for `contrib/openddlparser/code/OpenDDLParser.cpp` records comp_dir `/src/assimp`, and `addr2line -e assimp_fuzzer_vul 0x1230f00` returns line 551. The `/src/assimp/.git` HEAD is `575ce6136d4941a45c6ba894947d0e79fe032aff`, and its `OpenDDLParser.cpp` hashes to `f52809b5…` (identical to the extracted image source). Binary line numbering matches this source exactly.

Fix image, for completeness: HEAD is `3664fe20c07fdbd4d72c5caf68375b056806ab08` (merge PR #4146, 2021-10-29). `6a3ac62` is an ancestor of that HEAD, and its `OpenDDLParser.cpp` and `OpenDDLParserUtils.h` hashes match the Task 004 after-versions exactly.

Binary SHA-256: `397e121ddb42978a516f9afa35422cea30e018b390eb4af2cb6c3aab44040bb7`.

## Q4: Is evidence sufficient to safely choose a source baseline?

Evidence status:

- Binary ↔ image source (`575ce613`): fully confirmed via DWARF + addr2line + git hash.
- Fix image source ↔ official fix commit result: fully confirmed via hash equality on both `OpenDDLParser.cpp` and `OpenDDLParserUtils.h`.
- Official parent source (`10737165`) ↔ vul binary source (`575ce613`): NOT the same revision. `575ce613` is an ancestor separated by 5 commits that rewrote the target file.
- Official patch applicability to vul source: **fails** `git apply --check` at the first hunk.

Two baselines therefore exist and they are not interchangeable:

1. `575ce613` — matches the vulnerable binary byte-for-byte; line 551 is exact. Required if the experiment must reproduce the published ASan line numbers or be diffed against the shipped binary.
2. `10737165` — matches the official patch context; the official diff applies cleanly. Required if the experiment must use the official patch as-is.

A patch constructed against baseline 2 will not apply to baseline 1 without adaptation, and vice versa. This is a factual constraint, not a recommendation.
