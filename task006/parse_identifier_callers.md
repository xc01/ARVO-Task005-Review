# Task 006 parseIdentifier Callers

Source: `575ce6136d4941a45c6ba894947d0e79fe032aff` (vul image binary baseline).

Three direct call sites exist in `OpenDDLParser.cpp`:

## 1. parseHeader (line 268)

```c
262 char *OpenDDLParser::parseHeader( char *in, char *end ) {
263     if( ddl_nullptr == in || in == end ) { return in; }
267     Text *id( ddl_nullptr );
268     in = OpenDDLParser::parseIdentifier( in, end, &id );
```

- `in`/`end` passed directly from `parseNextNode`/`parse()`.
- No remaining-length check between entry and call beyond the initial `in == end` guard.
- **Not** the frame that crashes for P1: this first call returns normally after consuming `"emetric"` at the `$` terminator. It is the first half of P1's path, not the faulting entry.

Note on the upstream revision: in the vul image source the guard at line 263 is `in == end` only, and line 268 calls `parseIdentifier` with no bound re-check. The official patch later adds `in != end` guards at lines 296/299/320 in the *parent* source, but those lines do not exist in this revision.

## 2. parseName (line 523)

```c
504 char *OpenDDLParser::parseName( char *in, char *end, Name **name ) {
506     if( ddl_nullptr == in || in == end ) { return in; }
511     in = lookForNextToken( in, end );
512     if( *in != '$' && *in != '%' ) { return in; }
523     in = parseIdentifier( in, end, &id );
```

- Entry guard `in == end` exists, but line 512 dereferences `*in` after `lookForNextToken` without re-checking `in != end`; a secondary OOB risk exists in `parseName` itself.
- Once past the guard, `in++` at line 520 then `parseIdentifier(in, end, &id)` is called; if `in` was advanced to `end`, the callee's entry guard `in == end` catches it. If not, the same dangerous loop applies.
- **This is P1's actual faulting call site**: ASan frame #1 is `parseName ... OpenDDLParser.cpp:523:10`.
- For P1, `in++` advances from the `$` at index 7 to index 8 (the NUL appended by `TextFileToBuffer`), which is still `!= end`, so the callee guard does not fire and the dangerous loop is entered at index 8 and faults at index 9.
- Line 512 is itself an unrechecked dereference after `lookForNextToken`, so `parseName` has an independent single-byte OOB risk of its own; P1 does not exercise it.

## 3. parseProperty (line 861)

```c
853 char *OpenDDLParser::parseProperty( char *in, char *end, Property **prop ) {
855     if( ddl_nullptr == in || in == end ) { return in; }
859     in = lookForNextToken( in, end );
861     in = parseIdentifier( in, end, &id );
```

- Entry guard `in == end` exists before `lookForNextToken`, but after `lookForNextToken` returns, there is no `in != end` re-check before `parseIdentifier`. The callee's own entry guard catches `in == end`.
- Reachable from `parseHeader` line 301 via the property-parsing loop.
- Same dangerous loop in callee; structurally reachable but not confirmed by P1's current stack.
- In this revision the call is at line 861 and the property loop guarding it is line 299; the official patch's `in != end` additions at parent lines 296/299/320 have no counterpart here.

Only source-derived facts are reported; no alternate input was constructed or run.
