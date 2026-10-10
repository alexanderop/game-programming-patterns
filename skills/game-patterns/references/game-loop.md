# Game Loop

Source: https://gameprogrammingpatterns.com/game-loop.html

Run input, update, and render in a continuous loop that tracks real time, so game speed does not depend on input timing or processor speed.

## Use when

- You are building almost any game. Even turn-based games need a loop, because animation, audio, and effects keep running while the game waits for the player.
- You own the entry point. An engine hides this loop, but it still exists.

## Skip when

- An engine or framework already owns the loop and calls your update. Hook into it instead of writing a second one.
- The program is a plain event-driven app with no continuous simulation.

## Sketch

```ts
const STEP_MS = 1000 / 60
const MAX_STEPS_PER_FRAME = 5

class GameLoop {
  #last = performance.now()
  #lag = 0

  readonly #update: (dtMs: number) => void
  readonly #render: (alpha: number) => void

  constructor(update: (dtMs: number) => void, render: (alpha: number) => void) {
    this.#update = update
    this.#render = render
  }

  start(): void {
    requestAnimationFrame(this.#frame)
  }

  #frame = (now: number): void => {
    this.#lag += now - this.#last
    this.#last = now

    let steps = 0
    while (this.#lag >= STEP_MS && steps < MAX_STEPS_PER_FRAME) {
      this.#update(STEP_MS)
      this.#lag -= STEP_MS
      steps++
    }
    if (steps === MAX_STEPS_PER_FRAME) this.#lag = 0

    this.#render(this.#lag / STEP_MS)
    requestAnimationFrame(this.#frame)
  }
}
```

Update runs in fixed steps, rendering runs once per display frame, and `alpha` (0 up to just under 1) tells the renderer how far it is between the last two states so it can interpolate positions. The step cap prevents the spiral of death, where a slow frame demands more updates, which makes the next frame slower still. Dropping the leftover lag trades a visible hitch for staying responsive.

## Pitfalls

- Variable timesteps make physics unstable and gameplay non-deterministic, which also hurts networking and replays.
- Without interpolation, fixed updates at 60 Hz look jittery on a 120 Hz or irregular display.
- A tab that returns from the background delivers a huge time delta. Clamp it.
- Update must stay cheap enough to run well within one step on the slowest target. Otherwise the cap triggers constantly.
- Keep the step small enough for fast machines to feel smooth, but not so small that slow machines cannot keep up.

## Decisions

- Who owns the loop:
  - Platform loop such as `requestAnimationFrame`. This is simple and well integrated, but you get less control over timing.
  - Engine loop. It saves effort and limits control.
  - Your own loop. It gives full control, but you must hand time back to the platform so the page stays responsive.
- Timestep:
  - Fixed step without sync is the simplest, but speed depends on hardware.
  - Variable step adapts to hardware, but breaks determinism.
  - Fixed update with variable rendering is the most flexible and the most complex. Prefer it for anything with physics.
- Power use: running flat out is smooth but drains batteries, so cap the frame rate on mobile.

## Seen in

Every game engine, browser games driven by `requestAnimationFrame`, and Glenn Fiedler's "Fix Your Timestep" article.
