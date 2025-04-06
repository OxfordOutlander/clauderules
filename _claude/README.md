# Claude Knowledge Graph Directory

## Purpose

This directory contains structured documentation to help Claude understand and work effectively with the codebase. It serves as Claude's "long-term memory" for the project, providing comprehensive information about code structure, patterns, implementation details, and technical context.

## Directory Structure

- **MODEL_DOCUMENTATION.md**: Comprehensive project overview
- **memory_anchors.md**: Stable reference points in code
- **code_structure/**: Combined code relationships and metadata
- **patterns/**: Implementation pattern libraries
- **delta/**: Version-to-version changes during implementation sessions
- **enhancements/**: Planning documents for future features and improvements
- **troubleshooting/**: Problem-solution pairs and debugging history
- **tests/**: Validation and test scripts with test data
- **runs/**: Application execution logs

## Using This Knowledge Graph

When working with Claude, always point to specific files in this directory for context relevant to your task. For example:

```
Please review _claude/patterns/error_handling_pattern.md before helping me fix this error.
```

## Special Instructions for Claude

- Always check MODEL_DOCUMENTATION.md first for high-level understanding
- Use memory_anchors.md to locate important code sections
- Reference patterns/ before implementing new features
- Check troubleshooting/ when debugging issues
- Consult enhancements/ for planned improvement ideas
- Review delta/ to understand recent changes

## Maintenance Guidelines

- Update delta/ files after each implementation session
- Create enhancement plans before implementing significant features
- Add memory anchors for important new code sections
- Document new patterns as they emerge
- Create test summaries after implementing new features