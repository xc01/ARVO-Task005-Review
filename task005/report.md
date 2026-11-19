# Task 005 Report

## Runtime Findings

The vulnerability image produced an ASan report:

- Type: `heap-buffer-overflow`
- Access: `READ of size 1`
- Location: `OpenDDLParser.cpp:551:26`
- Function: `ODDLParser::OpenDDLParser::parseIdentifier`
- Call path: `parseIdentifier` -> `parseName` -> `parseHeader` -> `parseNextNode` -> `parse`
- Exit code: 1, no timeout.

The fix image used the identical `/tmp/poc`, completed normally with exit code 0, and did not emit an ASan report. This confirms execution on the same input and the absence of an observed memory error, not a semantic proof beyond the observed run.

## Static Source Facts

`OpenDDLParser.cpp` differed between images. The vulnerability image version hashes to `f52809b502c84231cae4a51121f8f35cf6925cfce1f4a806298591760b1f87d8`; the fix image version hashes to `7429fc762dd71f60f7618299e5a642189a60e75f93d34a760c40824f8f173a7b`.

## Unverified

- Exact Git commit inside each image.
- Whether the images correspond byte-for-byte to the expected official source commits.
- Semantic completeness of the official patch beyond the single observed run.
