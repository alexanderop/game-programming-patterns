---
type: llm
---
The answer is advice to a game developer. Judge only the final answer.

Pass only if all of these hold:
1. Bounds work per drain so chain reactions cannot hang the frame: processes only events present at drain start (rest wait until next frame), swaps two queues, or applies a hard cap.
2. Skips or validates events whose target entity already died or was removed earlier in the drain.
3. Code shows the queue and the drain loop.

Fail if the answer gives advice that would make the problem worse, or if it is cut off before giving a recommendation.
