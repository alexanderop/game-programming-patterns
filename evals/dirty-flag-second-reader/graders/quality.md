---
type: llm
---
The answer is advice to a game developer. Judge only the final answer.

Pass only if all of these hold:
1. Caches world matrices and marks a node dirty when its local transform changes.
2. worldPosition() returns a correct value when called before rendering in the same frame after a parent moved: either it checks and recomputes up the ancestor chain on read, or setting a local transform marks all descendants dirty, or an explicit update pass runs before physics. Propagating dirtiness only during the render traversal fails this criterion.
3. Unchanged nodes are not recomputed every frame.

Fail if the answer gives advice that would make the problem worse, or if it is cut off before giving a recommendation.
