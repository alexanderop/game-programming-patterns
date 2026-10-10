# Prototype

Source: https://gameprogrammingpatterns.com/prototype.html

An object spawns new objects like itself by cloning, copying both its type and its state. The book finds the class-based form rarely worth using, and the delegation form useful for data.

## Use when

- Game data is a property bag (JSON or similar) with many near-duplicates, such as goblin variants or unique items. Let an entry name a `prototype` and override only what differs.
- One-off entities like bosses are small tweaks of a common base.
- You want a spawner for each variant (fast ghost, weak ghost) without a spawner class per variant.

## Skip when

- A plain spawn function or a class reference does the job. In TypeScript, classes and functions are values, so you can pass them around directly.
- You would write a `clone()` for every class. That costs about as much code as the spawners it replaces, and deep versus shallow copying is easy to get wrong.
- You are modeling kinds of things. Prefer components or a type object.

## Sketch

```ts
type EntityData = {
  readonly prototype?: string
  readonly name?: string
  readonly health?: number
  readonly weaknesses?: readonly string[]
}

type Resolved = { name: string; health: number; weaknesses: readonly string[] }

function resolve(id: string, db: Readonly<Record<string, EntityData>>): Resolved {
  const data = db[id]
  if (!data) throw new Error(`unknown entity ${id}`)
  const base = data.prototype ? resolve(data.prototype, db) : { name: id, health: 1, weaknesses: [] }
  return {
    name: data.name ?? base.name,
    health: data.health ?? base.health,
    weaknesses: data.weaknesses ?? base.weaknesses,
  }
}

const db = {
  goblin: { name: 'Goblin', health: 20, weaknesses: ['fire'] },
  goblinArcher: { prototype: 'goblin', health: 12 },
} as const
```

The archer inherits name and weaknesses from the goblin and overrides health, so shared values live in one place. No abstract base goblin is needed.

## Pitfalls

- Cyclic prototype chains loop forever. Validate data at load time.
- Resolve once at load, not on every lookup.
- Changing a prototype changes every entity that delegates to it. That is the point, but it surprises content authors.
- A whole language built on prototypes, with no classes, tends to rebuild class-like structure in libraries. The author found it unpleasant to work in.

## Decisions

- Clone or delegate: cloning copies values up front. Delegation looks up missing fields in the parent at read time, or at load time as above.
- Shallow or deep copy: decide whether a cloned object owns copies of its parts or shares them.
- Single or multiple parents: one parent keeps resolution simple.

## Seen in

JavaScript's prototype chain, the Self language, and data-driven monster and item files with an inheritance field.
