# Memory Anchors

This document lists memory anchors that should be added to the codebase to assist in navigation and reference.

## How to Use Memory Anchors

Memory anchors are special comments in key files that serve as stable reference points. They help pinpoint exact locations in the code that might change over time.

Format: `# [ANCHOR-NAME] <description> [UUID]`

Example: `# [ANCHOR-MODULE-NAME] Module description [module-uuid-1234]`

## Anchors to Add

[List your anchor points here]

## Using Anchors in Documentation

When referring to code sections in documentation, use the anchor name and UUID. For example:

"For more details on this module, see [ANCHOR-MODULE-NAME] in path/to/file.js."

This allows precise reference even if line numbers change over time.