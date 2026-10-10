# game-programming-patterns

A Claude Code plugin that gives coding agents the patterns from Robert Nystrom's [Game Programming Patterns](https://gameprogrammingpatterns.com) as a skill, so they choose a sensible architecture when building games.

The skill holds a problem-to-pattern table and one short reference per pattern. Each reference covers when to use the pattern, when to skip it, a TypeScript sketch, pitfalls, design decisions, and a link to the book chapter. The agent loads only the references it needs.

## Install

```sh
claude plugin marketplace add alexanderop/game-programming-patterns
claude plugin install game-programming-patterns@game-programming-patterns
```

## Patterns

- Design patterns revisited: command, flyweight, observer, prototype, singleton, state
- Sequencing: double-buffer, game-loop, update-method
- Behavioral: bytecode, subclass-sandbox, type-object
- Decoupling: component, event-queue, service-locator
- Optimization: data-locality, dirty-flag, object-pool, spatial-partition

## Evals

`evals/` holds 5 hard cases for `claude plugin eval`. Each is a feature request with a trap the book warns about (stale pool handles, catch-up spiral, listeners removed during emit, unbounded event chains, stale cached transforms). An LLM judge checks whether the answer handles the trap without being told.

```sh
claude plugin eval . --model claude-haiku-5-5 --judge-model claude-sonnet-5-5 --runs 3
```

Baseline on Claude Haiku 5.5, 3 runs per case: 15/15 with the plugin, 14/15 without, skill loaded in 15/15 runs, about $1.24 per run. Current models already know these patterns for single questions, so the cases work as a regression check for triggering and content, not as proof of a quality gain.

## Credit

All ideas come from "Game Programming Patterns" by Robert Nystrom. The references are summaries written for agents. Read the book for the full reasoning and examples, and buy a copy if it helps you.
