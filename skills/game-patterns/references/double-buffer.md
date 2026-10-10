# Double Buffer

Source: https://gameprogrammingpatterns.com/double-buffer.html

Keep two copies of state. Readers see the current copy, writers fill the next copy, and a swap makes the finished work visible all at once.

## Use when

- State changes in many small steps and other code may read it partway through. Example: entities update one after another, but each should see the previous frame of its neighbors.
- Readers must never see half-finished work, and must not wait for the writer.
- Update order would otherwise change the outcome, such as two actors that both react to each other in the same frame.

## Skip when

- Nothing reads the state while it changes. There is no tearing to hide.
- Memory is tight and the state is large. You pay for two copies.
- Copying or swapping costs more than the modification itself.

## Sketch

```ts
interface Cell {
  readonly alive: boolean
}

class Grid {
  #current: Cell[]
  #next: Cell[]

  constructor(readonly width: number, readonly height: number) {
    const make = () => Array.from({ length: width * height }, (): Cell => ({ alive: false }))
    this.#current = make()
    this.#next = make()
  }

  read(x: number, y: number): Cell {
    return this.#current[y * this.width + x]!
  }

  write(x: number, y: number, cell: Cell): void {
    this.#next[y * this.width + x] = cell
  }

  swap(): void {
    ;[this.#current, this.#next] = [this.#next, this.#current]
  }
}
```

Every `read` goes to the old frame and every `write` goes to the new one, so the order of cell updates cannot matter. Call `swap` once per frame after all writes, and note that `#next` still holds data from two frames ago until you overwrite it.

## Pitfalls

- The swap must be atomic. Nothing may read or write either buffer while it runs.
- After a pointer swap, the buffer you now write into holds stale data. Write every slot each frame, or copy first.
- Effects now arrive one frame late. That is the point, but tests and designers will notice it.
- Callers cannot hold long-lived references into a buffer, because the reference points at the wrong copy after a swap.

## Decisions

- How to swap:
  - Swap the references. This is nearly free, but the data is two frames old and external holders of a buffer break.
  - Copy one buffer into the other. The data stays one frame old and addresses stay fixed, but the cost grows with buffer size.
- Granularity:
  - One buffer pair for the whole structure needs a single swap and is the simplest option.
  - A pair per object means the swap must visit every object. A shared "current index" that each object uses to pick its slot turns this into one flip.

## Seen in

Framebuffers and swap chains in graphics APIs, cellular automata such as Game of Life, simultaneous-move simulations, and physics or AI steps that must not see their own partial results.
