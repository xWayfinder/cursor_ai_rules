# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records (ADRs) for this project.

## What are ADRs?

Architecture Decision Records are short text documents that capture important architectural decisions made during the development of a software project. They provide context for why certain decisions were made and help future developers understand the reasoning behind the current architecture.

This ADR format follows the [adr-tools](https://github.com/npryce/adr-tools) standard, which is based on Michael Nygard's original ADR format.

## ADR Format

Each ADR follows this structure:

- **Title**: A descriptive title
- **Status**: Proposed, Accepted, Deprecated, Superseded, etc.
- **Context**: The problem or situation that led to this decision
- **Decision**: The decision that was made
- **Consequences**: The resulting context after applying the decision

## ADR Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](0001-record-architecture-decisions) | Record Architecture Decision | Accepted |

## Creating a New ADR

1. Run: `adr new My new decisions`
2. Fill in the context, decision, and consequences
3. Update this README with the new ADR entry
4. Commit the changes

## References

- [adr-tools](https://github.com/npryce/adr-tools) - Command-line tools for working with ADRs
- [ADR GitHub Repository](https://github.com/joelparkerhenderson/architecture_decision_record)
- [ADR Wikipedia](https://en.wikipedia.org/wiki/Architecture_decision_record)

## Installing adr-tools (Optional)

To use the adr-tools command-line utilities, see the [installation instructions](https://github.com/npryce/adr-tools/blob/master/INSTALL.md) in the adr-tools repository.