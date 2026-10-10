# Type Object

Source: https://gameprogrammingpatterns.com/type-object.html

Model many kinds of things with one class. Each instance points at a shared type object that holds the values that define its kind.

## Use when

- You have many kinds of entities that differ mostly in data. Example: a hundred enemy species that differ in health, speed, and sprite.
- Designers or mod authors should add or tune kinds without a code change or rebuild.
- You do not know the full list of kinds up front, for example when downloaded content adds new ones.

## Skip when

- The kinds differ in behavior more than data. Type objects hold values well and logic poorly.
- You have three or four kinds that never change. A plain union or enum is simpler.

## Sketch

```ts
interface MonsterType {
  readonly name: string
  readonly maxHealth: number
  readonly attack: string
  readonly parent?: MonsterType
}

const resolve = <K extends keyof MonsterType>(t: MonsterType, key: K): MonsterType[K] =>
  t[key] ?? (t.parent ? resolve(t.parent, key) : t[key])

class Monster {
  #health: number

  constructor(readonly type: MonsterType) {
    this.#health = resolve(type, 'maxHealth')
  }

  get attack(): string {
    return resolve(this.type, 'attack')
  }
}

const types = new Map<string, MonsterType>() // filled from a JSON file at load time
```

The type table is loaded from data, so adding a species is a data edit. The optional `parent` gives single inheritance between types, so a troll variant overrides only what differs.

## Pitfalls

- You now own the bookkeeping. Create, store, and keep alive every type object, and make sure no instance holds a stale or missing one.
- Type-specific behavior does not fit in data. Let data choose among a fixed set of coded behaviors, or add a scripting layer, and accept the runtime cost.
- Forwarding every type field through the instance bloats the instance API. Decide early how much to hide.

## Decisions

- Encapsulated or exposed type object:
  - Encapsulated hides the pattern and lets an instance override type data, but needs forwarding methods.
  - Exposed lets code use a type without an instance, but widens the public API.
- Construction: the caller builds the instance, or a factory on the type object does. A factory helps with pooling.
- Can an instance change type at runtime? Fixed is easier to debug. Mutable avoids reallocation, but you must recheck the instance against the new type's assumptions.
- Inheritance between types:
  - None duplicates values across similar types.
  - Single is usually the sweet spot. Resolve on lookup, or copy values down at load if they never change.
  - Multiple removes duplication but is hard to reason about.
- Types in code or in data. Data is easier for designers and less expressive.

## Seen in

Monster and item definitions loaded from JSON or spreadsheets, tile and block registries, unit stat tables in strategy games.
