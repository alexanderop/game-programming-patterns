---
name: pool-homing-missiles
tags: [apply]
max_turns: 10
timeout_seconds: 300
allowed_tools: [Skill, Read]
---
Add an object pool for enemies in my TypeScript shooter. Enemies have position, hp, a hit-flash timer, and a `target` field. Homing missiles store the enemy they're chasing. Enemies die often and new ones spawn constantly. Short answer with code.
