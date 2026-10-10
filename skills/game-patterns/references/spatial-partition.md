# Spatial Partition

Source: https://gameprogrammingpatterns.com/spatial-partition.html

Store objects in a structure keyed by position, so a query for neighbors touches only nearby objects instead of every pair.

## Use when

- Many objects have positions and you run proximity queries each frame. Example: collisions, targeting, area damage, separation steering.
- Checking all pairs is O(n squared) and a profiler shows it hurts.
- The world is large compared to the query radius.

## Skip when

- There are only dozens of objects. Brute force is simpler and often faster.
- Queries are rare or already limited by another rule, such as a small fixed list of enemies per room.
- Nearly everything is in range of everything else.

## Sketch

```ts
interface Body {
  id: number
  x: number
  y: number
}

class UniformGrid {
  readonly #cells = new Map<number, Body[]>()

  constructor(readonly cellSize: number, readonly cols: number) {}

  #key(x: number, y: number): number {
    return Math.floor(y / this.cellSize) * this.cols + Math.floor(x / this.cellSize)
  }

  clear(): void {
    this.#cells.clear()
  }

  insert(b: Body): void {
    const k = this.#key(b.x, b.y)
    const cell = this.#cells.get(k)
    if (cell) cell.push(b)
    else this.#cells.set(k, [b])
  }

  *near(x: number, y: number): Generator<Body> {
    const cx = Math.floor(x / this.cellSize)
    const cy = Math.floor(y / this.cellSize)
    for (let dy = -1; dy <= 1; dy++) {
      for (let dx = -1; dx <= 1; dx++) {
        const cell = this.#cells.get((cy + dy) * this.cols + cx + dx)
        if (cell) yield* cell
      }
    }
  }
}
```

Rebuilding the grid each frame (`clear`, then `insert` every body) is the simplest way to handle moving objects. `near` returns candidates from the 3 by 3 block of cells, so the caller still does the exact distance test.

## Pitfalls

- Cell size must be at least the query radius, or the 3 by 3 scan misses targets.
- Cells that are too small make iteration overhead dominate. Cells that are too large put too many bodies in one bucket.
- Bodies with extent, not just a point, can span several cells. Insert them into each cell they overlap or test using their bounds.
- Clustered objects overload one cell, which is the weakness of fixed boundaries.
- Per-frame rebuilds allocate arrays and can cause GC churn. Reuse the arrays, or pair this with a pool.
- Pair checks visit each pair twice unless you compare ids or scan only half the neighbors.

## Decisions

- Flat or hierarchical. A flat grid is simple, cheap to update, and uses fixed memory. A quadtree adapts to empty space and dense clusters but costs more code and update work.
- Fixed or adaptive boundaries. Fixed cells suit moving objects. Adaptive trees (BSP, k-d) balance better but suit static geometry built once.
- Rebuild or update in place. Rebuild is simplest when most things move. Incremental moves, which relink only bodies that cross a cell boundary, win when most things sit still.
- Partition only or a separate list too. A separate list helps when you must visit every object regardless of location.

## Seen in

Collision broad phases, AI target lookup, area-of-effect queries, boids neighbor search, tile and chunk worlds.
