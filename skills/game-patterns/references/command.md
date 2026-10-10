# Command

Source: https://gameprogrammingpatterns.com/command.html

Turn a method call into an object. Once an action is a value, you can store it, queue it, send it over a network, replay it, or undo it.

## Use when

- Players rebind controls. Map each button to a command object instead of hard-wiring input to actions.
- The same action must drive different actors. AI, replays, and demo modes can emit the commands a player would.
- You need undo and redo, as in a turn-based game or a level editor.
- You need replays or lockstep networking. Recording each entity's commands per frame is far cheaper than recording full state.

## Skip when

- A direct call is clear enough. Do not wrap every method in a command.
- Undo is wanted but some state changes bypass commands. Undo breaks as soon as one write skips the command path.

## Sketch

```ts
interface Command {
  execute(): void
  undo(): void
}

class MoveUnit implements Command {
  readonly #unit: Unit
  readonly #to: Vec2
  #from: Vec2 | undefined

  constructor(unit: Unit, to: Vec2) {
    this.#unit = unit
    this.#to = to
  }

  execute(): void {
    this.#from = this.#unit.position
    this.#unit.position = this.#to
  }

  undo(): void {
    if (this.#from) this.#unit.position = this.#from
  }
}

class History {
  readonly #done: Command[] = []
  #cursor = 0

  run(cmd: Command): void {
    this.#done.length = this.#cursor // drop the redo tail
    cmd.execute()
    this.#done.push(cmd)
    this.#cursor++
  }
  undo(): void {
    if (this.#cursor > 0) this.#done[--this.#cursor]!.undo()
  }
}
```

The command captures the previous position inside `execute`, so it records exactly what it changed. Running a new command after an undo discards the redo tail. Redo steps the cursor forward and calls `execute` again.

## Pitfalls

- Undoable commands bind to one target and one change. They are not reusable "move something" verbs.
- Unbound inputs need a do-nothing command (a null object) so callers skip null checks.

## Decisions

- Reusable or specific commands:
  - Reusable commands take the actor as a parameter to `execute`, so one instance serves any actor.
  - Specific commands hold their target and arguments, which suits queues and undo.
- Class or closure: use a class when a command has several operations such as `execute` and `undo`. Use a closure for a single fire-and-forget action.

## Seen in

Rebindable input, AI controlling units, replay recording, lockstep multiplayer, and undo in strategy games and editors.
