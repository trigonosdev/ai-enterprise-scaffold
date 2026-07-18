# ADR-0001: Record Architecture Decisions

**Status:** Accepted
**Date:** [YYYY-MM-DD — set this to the date you actually adopt this scaffold]

## Context

This project is built with heavy AI assistance. Each AI session starts
with no memory of prior sessions' reasoning — only the code and docs
those sessions left behind. Without a durable record of *why* a
non-obvious decision was made, later sessions (AI or human) tend to
either re-litigate settled questions or "helpfully" revert deliberate,
unusual-looking choices back to the naive default.

## Decision

We will use Architecture Decision Records (ADRs), as described by
Michael Nygard, to record every significant architectural decision made
on this project. See [`README.md`](README.md) in this folder for the
process rules, and [`template.md`](template.md) for the format every new
ADR follows.

## Why

The alternative — relying on commit messages, PR descriptions, or a wiki
page — doesn't hold up under context-reset drift the way a permanent,
append-only, numbered record does. A commit message is easy to miss when
searching; a wiki page is editable, so its history of *changed reasoning*
is easy to lose. ADRs are deliberately immutable once merged (see rule 2
in this folder's README) specifically so that "we tried X and rejected
it, here's why" stays discoverable even after a later ADR supersedes it.

## Consequences

**Easier:** any future session — AI or human — can answer "why does this
look the way it does" by reading a short, dated, specific document,
instead of re-deriving the reasoning or guessing.

**Harder:** every significant decision now has a small amount of
mandatory documentation overhead. This is a deliberate trade — the cost
is paid once, by whoever made the decision, while the benefit is paid out
to every future session that would otherwise have to reconstruct the
reasoning from scratch.

## Status Notes

*(None yet — this ADR hasn't been superseded.)*
