# Task 006 P1 Analysis

## Raw content

| property | value |
|---|---|
| hex | `65 6d 65 74 72 69 63 24` |
| ASCII | `emetric$` |
| SHA-256 | `699ec24176021082db0d9a6b51bfca800af79441c444a429122f9ec94b5967f0` |
| size on disk | 8 bytes |

No byte was modified. Both images carried this identical file.

## Where the 9-byte heap region comes from

ASan reports: `0x602000000239 is located 0 bytes to the right of 9-byte region [0x602000000230,0x602000000239)`.

The file is 8 bytes, so the 9th byte is not from the PoC. `BaseImporter::TextFileToBuffer` (code/Common/BaseImporter.cpp:443-467) always appends a NUL:

```c
455    data.reserve(fileSize + 1);
456    data.resize(fileSize);
458    if (fileSize != stream->Read(&data[0], 1, fileSize)) { throw ...; }
462    ConvertToUTF8(data);
466    data.push_back(0);      // "append a binary zero to simplify string parsing"
```

So `OpenGEXImporter::InternReadFile` (OpenGEXImporter.cpp:309-313) hands a **9-byte** buffer to `OpenDDLParser::setBuffer`, which is exactly the allocation ASan attributes to `OpenDDLParser.cpp:179` (`m_buffer.resize(len)`).

Buffers involved:

- before `normalizeBuffer`: 9 bytes `e m e t r i c $ \0`
- `normalizeBuffer` copies the same 9 bytes into `newBuffer` (no newline, no comment markers), then `buffer = newBuffer`
- the resulting `m_buffer` occupies the same 9-byte allocation

## Boundary pointer

```c
223    char *current( &m_buffer[ 0 ] );
224    char *end( &m_buffer[m_buffer.size() - 1 ] + 1 );
```

With size 9: `current = index 0`, `end = index 9` (one past the last valid byte). The last *valid* byte, index 8, is the NUL.

## Call path actually taken

The ASan stack, frame by frame:

| frame | function | location | role |
|---|---|---|---|
| #0 | `parseIdentifier` | `OpenDDLParser.cpp:551:26` | faulting dereference |
| #1 | `parseName` | `OpenDDLParser.cpp:523:10` | **actual caller at crash** |
| #2 | `parseHeader` | `OpenDDLParser.cpp:286:8` | caller of `parseName` |
| #3 | `parseNextNode` | `OpenDDLParser.cpp:246:10` | |
| #4 | `parse` | `OpenDDLParser.cpp:227:19` | main loop |
| #5 | `OpenGEXImporter::InternReadFile` | `OpenGEXImporter.cpp:314:28` | `myParser.parse()` |

Note that `parseHeader`'s *own* `parseIdentifier` call at line 268 is frame-less here because it completed successfully; the crash happens on the second entry, made from `parseName`.

## Step-by-step state trace (static)

1. `parseHeader(in=0, end=9)`: guard `in != end` passes.
2. line 268 `parseIdentifier(0, 9)`: sees `e`, not numeric, enters the loop at 551.
3. Loop advances over `e m e t r i c` (indices 0-6). At index 7 the byte is `$`, and line 555 `*in != '$'` is false, so the loop **exits normally**. `id = Text(0, 7) = "emetric"`, return `in = 7`.
4. `parseHeader` line 274 `lookForNextToken(7, 9)`: `$` is not a space, unchanged.
5. line 275-283: node created from `"emetric"`, `id` deleted.
6. line 286 `parseName(7, 9)`: guard passes; `lookForNextToken` leaves `in = 7`.
7. line 512 `*in == '$'` -> true, so parsing continues.
8. line 520 `in++` -> `in = 8`, which is the appended NUL. Still `!= end`.
9. line 523 `parseIdentifier(8, 9)`: guard `in == end` is false, so it proceeds.
10. line 541 `lookForNextToken`: NUL is not a space, unchanged, `in = 8`.
11. line 544 `isNumeric('\0')` false.
12. line 551 evaluated for `in = 8`: `isSeparator('\0')` false, `isNewLine('\0')` false, `in != end` true, `*in != '('`, `!= ')'`, `!= '$'` all true -> `++in` -> `in = 9`, `idLen = 1`.
13. line 551 evaluated for `in = 9`: **`isSeparator(*in)` dereferences index 9 before `in != end` is tested** -> 1-byte heap read past the 9-byte region. This is the reported fault at column 26.

## Exact offending condition

```c
551    while( !isSeparator( *in ) && 
552            !isNewLine( *in ) && ( in != end ) && 
553            *in != Grammar::OpenPropertyToken[ 0 ] &&
554            *in != Grammar::ClosePropertyToken[ 0 ] && 
555            *in != '$' ) {
```

`(in != end)` is the **third** conjunct. Short-circuit evaluation therefore performs two dereferences of `*in` while `in == end` before the bound check is ever consulted. Any path that leaves `in` exactly at `end` enters the loop body and then reads out of bounds on the following iteration test.

For P1, `in` reaches `end` in a single step from the appended NUL, because the NUL is neither a separator, nor a newline, nor a property/name token. The trailing `$` in the PoC is what drives execution into `parseName` and past `in++`, which is why it is load-bearing rather than filler.

This is static reasoning over the shipped source plus the published ASan trace. No new input was executed.
