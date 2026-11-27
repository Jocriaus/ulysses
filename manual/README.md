# Making a game in Godot: the Ulysses manual

This manual explains how **Ulysses** (the flood rescue game in `ulysses-godot/`) was built with **Godot 4.7** and **GDScript**, part by part. It is written for two readers at once:

- **A web backend developer** who knows variables, classes, events, config files, tests and deployment, but has never made a game. Every chapter maps Godot ideas onto things you already know (services, pub/sub, request loops, migrations, CI).
- **A gamer** who knows what a good game *feels* like and wants to know how that feel is made: why the boat eases into motion, why the flood never walls you in, why a boss telegraphs its attack.

Each part of the game is described the same way:

1. **What it does** in play.
2. **Which Godot feature** makes it work (a node type, a signal, a shader, a resource).
3. **How the code works**, with the real code from this project.
4. **Alternatives**, and **why this one** was chosen.

## Chapters

| # | Chapter | You will learn |
|---|---|---|
| 1 | [Godot for a backend developer](01-godot-for-backend-developers.md) | The mental model: nodes, scenes, the game loop, signals, resources, autoloads, mapped to backend ideas |
| 2 | [Getting around Godot](02-getting-around-godot.md) | Installing, the editor, opening this project, running, debugging, exporting to Android |
| 3 | [Learning GDScript](03-learning-gdscript.md) | The language through this game's code, a study path, and the traps we actually hit |
| 4 | [The architecture of Ulysses](04-architecture.md) | Every scene and script, how data flows, and why it is organised this way |
| 5 | [The boat and the core rules](05-boat-and-core-rules.md) | Movement, momentum, the current, fuel, crushing, pickups, carrying people, score, Continue |
| 6 | [The spawn director](06-spawn-director.md) | Procedural generation: rows, a guaranteed safe path, shuffle bags, patterns, waves, seeds |
| 7 | [Bosses](07-bosses.md) | Attack patterns, bullets, reachability maths, telegraphs, wind, crates |
| 8 | [UI, menus and progression](08-ui-and-progression.md) | Control nodes, themes, the HUD, shop, customize, missions, the Daily, the tutorial |
| 9 | [Visuals](09-visuals.md) | Shaders (water, sprite effects), particles, tweens, sprites and frames, the camera, the Aseprite pipeline |
| 10 | [Audio](10-audio.md) | An audio service, buses, music and sound effects |
| 11 | [Saving, phones and shipping](11-saving-and-mobile.md) | ConfigFile saves, app lifecycle, touch input, vibration, Android export |
| 12 | [Testing and tooling](12-testing-and-tooling.md) | Headless tests, a sandboxed save, screenshots, deterministic seeds |
| 13 | [Where to go next](13-next-steps.md) | Exercises on this codebase and a learning plan |

## How to read it

- **New to Godot?** Read 1, 2 and 3 in order, then open the project and keep it open while you read 4.
- **Want one mechanic?** Jump to its chapter; each one stands on its own and links back where it needs to.
- **Learning by doing?** Chapter 13 has small changes to make in this codebase, from "change a number" to "add a boss".

Every file path in this manual is relative to `ulysses-godot/` unless it says otherwise, and every number comes from `scripts/tuning.gd`, the one place the game's balance lives.
