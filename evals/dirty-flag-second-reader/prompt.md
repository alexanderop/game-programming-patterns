---
name: dirty-flag-second-reader
tags: [apply]
max_turns: 10
timeout_seconds: 300
allowed_tools: [Skill, Read]
---
My TypeScript scene graph has about 5000 nodes (ships, turrets on ships, crew on turrets) and I recompute every world matrix every frame by walking the parent chain. Most nodes never move. Add caching so unchanged nodes don't get recomputed. Physics and AI targeting call node.worldPosition() during their update, before rendering. Short answer with code.
