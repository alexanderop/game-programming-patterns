---
name: event-queue-chains
tags: [apply]
max_turns: 10
timeout_seconds: 300
allowed_tools: [Skill, Read]
---
Design an event queue for gameplay events in my TypeScript game (damage, explode, die). The queue is drained once per frame. Handlers can push new events, for example an explode handler pushes damage events for nearby enemies, which can cause more explosions. Short answer with code.
