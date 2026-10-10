# Bytecode

Source: https://gameprogrammingpatterns.com/bytecode.html

Encode behavior as instructions for a small virtual machine, so designers and modders can change it as data and the VM sandboxes it from the engine.

## Use when

- The game has a lot of behavior to define, and the host language is too low-level, too slow to iterate in, or too trusted for the people writing it.
- Untrusted content such as mods must not crash the game or touch things it should not.
- Behavior must load at runtime, save to disk, or travel over the network.

## Skip when

- The code is performance-critical engine work. Bytecode runs slower than native code.
- A few configurable parameters or a plain data table would do the job.
- You have no plan for authoring tools. Raw bytecode is not something people write by hand.

## Sketch

```ts
type Op =
  | { readonly kind: 'push'; readonly value: number }
  | { readonly kind: 'add' }
  | { readonly kind: 'setHealth'; readonly wizard: number }
  | { readonly kind: 'playSound'; readonly id: number }

interface Host {
  setHealth(wizard: number, amount: number): void
  playSound(id: number): void
}

function run(program: readonly Op[], host: Host): void {
  const stack: number[] = []
  const pop = (): number => {
    const v = stack.pop()
    if (v === undefined) throw new Error('stack underflow')
    return v
  }
  for (const op of program) {
    switch (op.kind) {
      case 'push': stack.push(op.value); break
      case 'add': stack.push(pop() + pop()); break
      case 'setHealth': host.setHealth(op.wizard, pop()); break
      case 'playSound': host.playSound(op.id); break
    }
  }
}
```

Instructions that touch the game go through `Host`, which is the only door out of the sandbox. Everything else works on the stack. In TypeScript a discriminated union of ops is often enough, and packing to real bytes only pays off when size or speed demands it.

## Pitfalls

- Scope creep is the main danger. A small script system grows into a full language by accident. Decide up front how far it goes.
- You need a front end, either a compiler or a graphical editor. Without one, nobody can author programs.
- Debugging gets harder, because your debugger shows VM state and not the script. Plan for tracing and error messages that point at the source.
- Bound resources. Cap the stack size, count executed instructions, and omit jumps if scripts must always terminate.
- Validate programs before running them when they come from outside, especially with untagged values.

## Decisions

- Stack or register VM:
  - Stack machines have small instructions and are easier to generate code for, but need more of them. Start here.
  - Register machines have larger instructions and need fewer, and the gain depends on the instruction set.
- Instruction groups: external primitives that reach the game, internal primitives such as literals and math, control flow such as jumps, and abstraction such as callable procedures.
- Value representation:
  - A single type is simplest but limiting.
  - A tagged union is safe and supports dynamic types, at some memory cost.
  - An untagged union is compact but unsafe without verification.
  - An object interface is open-ended but slow.
- Authoring:
  - A text language needs a syntax, a parser, and good error messages.
  - A graphical tool prevents many errors but costs UI work.

## Seen in

Lua and other embedded scripting VMs, the JVM, and visual scripting tools such as Unreal's Kismet.
