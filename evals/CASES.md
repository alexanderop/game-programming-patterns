# Eval cases

5 hard cases. Each is a feature request with a trap the user has not hit yet. A case passes only if the answer handles the trap without being told about it. Prompts never name the pattern or the trap.

| id | pattern | trap the answer must foresee |
|---|---|---|
| dirty-flag-second-reader | dirty flag | systems that read world transforms outside the render pass get stale values |
| game-loop-replays | game loop | catch-up spiral after a long frame or tab switch, and jitter without interpolation |
| pool-homing-missiles | object pool | missiles holding references to enemies that get released and reused, and leftover state |
| emitter-one-shot | observer | one-shot listeners removing themselves during emit, and lapsed listeners across restarts |
| event-queue-chains | event queue | handler chains that never finish, and events for entities that died earlier in the drain |

## dirty-flag-second-reader

````text
My TypeScript scene graph has about 5000 nodes (ships, turrets on ships, crew on turrets) and I recompute every world matrix every frame by walking the parent chain. Most nodes never move. Add caching so unchanged nodes don't get recomputed. Physics and AI targeting call node.worldPosition() during their update, before rendering. Short answer with code.
````

## game-loop-replays

````text
Write the main loop for my TypeScript canvas platformer. Physics must be deterministic because I record inputs for replays, and the game should look smooth on 60 Hz and 144 Hz monitors. Short answer with code.
````

## pool-homing-missiles

````text
Add an object pool for enemies in my TypeScript shooter. Enemies have position, hp, a hit-flash timer, and a `target` field. Homing missiles store the enemy they're chasing. Enemies die often and new ones spawn constantly. Short answer with code.
````

## emitter-one-shot

````text
Write a small typed event emitter for my TypeScript game. UI screens, achievements, and tutorial hints subscribe to gameplay events. Some tutorial hints should fire only once. Levels restart often and screens open and close a lot. Short answer with code.
````

## event-queue-chains

````text
Design an event queue for gameplay events in my TypeScript game (damage, explode, die). The queue is drained once per frame. Handlers can push new events, for example an explode handler pushes damage events for nearby enemies, which can cause more explosions. Short answer with code.
````
