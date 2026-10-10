---
type: llm
---
The answer is advice to a game developer. Judge only the final answer.

Pass only if all of these hold:
1. Reuses enemy objects through a pool instead of allocating per spawn.
2. Fully resets reused enemies, including the hit-flash timer and target.
3. Prevents missiles from chasing a recycled enemy: uses a generation counter or id check on the stored reference, clears missile targets when an enemy is released, or validates the target before use. An answer where a missile can follow a reused object to a new enemy fails.

Fail if the answer gives advice that would make the problem worse, or if it is cut off before giving a recommendation.
