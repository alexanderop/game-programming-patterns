---
type: llm
---
The answer is advice to a game developer. Judge only the final answer.

Pass only if all of these hold:
1. Uses a fixed simulation step with an accumulator so physics is deterministic.
2. Guards against the catch-up spiral after a long frame or tab switch by clamping the frame delta or capping updates per frame.
3. Renders with interpolation between the previous and current simulation state using the leftover fraction (alpha), so motion is smooth on high refresh displays.

Fail if the answer gives advice that would make the problem worse, or if it is cut off before giving a recommendation.
