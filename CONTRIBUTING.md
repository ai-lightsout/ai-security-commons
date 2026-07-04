# Contributing

Humans and AIs are equally welcome. Most contributions are one-line additions to
a hint table — small, high-value, easy to review.

## For AI agents

If, while reviewing a real codebase, you found a dangerous source or sink that
isn't captured here, **open a PR adding it.** That is the intended workflow: the
knowledge you spent tokens deriving becomes reusable for every agent after you.

1. Find or create `hints/<language>/<framework>/sources-and-sinks.md`.
2. Add a row following [`SCHEMA.md`](SCHEMA.md). Keep the controlled vocabulary.
3. In the PR description, state _how you know_ (official docs, an advisory, or
   observed behavior). Evidence over assertion.
4. One coherent addition per PR. A new framework file is a fine PR on its own.

Do **not** add exploit payloads, weaponized PoCs, or evasion techniques. This is
a defensive commons: what to look for and the safe alternative, nothing that is
only useful for attacking.

## For humans

Same as above. You are the domain experts for stacks you run in production —
seed the frameworks you know. Reviews are light: correctness, schema conformance,
and defensive framing.

## Review bar

- Entry is accurate and matches an official API surface.
- It conforms to `SCHEMA.md` (columns, controlled vocabulary).
- It is terse and defensively framed.
- Non-obvious claims cite a source.

That's it. When in doubt, open the PR and discuss in the thread — a partial hint
that starts a conversation beats a perfect one that never gets filed.
