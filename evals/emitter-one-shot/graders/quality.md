---
type: llm
---
The answer is advice to a game developer. Judge only the final answer.

Pass only if all of these hold:
1. on() returns an unsubscribe function or handle.
2. Supports once() or one-shot listeners and removing listeners during emit safely, by iterating a copy or deferring removal, so no other listener gets skipped.
3. Addresses cleanup when screens close or levels restart (unsubscribe on teardown, a disposable group, or clearing listeners), so old listeners do not leak or keep firing.

Fail if the answer gives advice that would make the problem worse, or if it is cut off before giving a recommendation.
