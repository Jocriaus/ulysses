# 1. Godot for a backend developer

If you build APIs, you already know most of the ideas a game engine uses. They just have different names and run 60 times a second instead of once per request.

## The big mapping

| Backend idea | Godot idea | In Ulysses |
|---|---|---|
| A request handler that runs per request | `_process(delta)` / `_physics_process(delta)` that run every frame | `boat.gd` moves the boat 60 times a second |
| An object graph / component tree | The **scene tree** of **nodes** | `Main` → `Boat`, `Spawners`, `Drifters`, `HUD`… |
| A reusable component or template | A **scene** (`.tscn`), instanced like a class | `boat.tscn`, `car_red.tscn`, `boss.tscn` |
| Events / pub-sub / observers | **Signals** (`signal`, `emit()`, `connect()`) | `boat.sunk` tells `game.gd` the run ended |
| A singleton service (logger, cache) | An **autoload** | `Audio` plays every sound in the game |
| Config files, data fixtures | **Resources** (`.tres`) | `hud_theme.tres`, `people.tres`, the shader materials |
| A database | A save file on the device (`ConfigFile`) | `user://save.cfg` |
| Environment config | `project.godot` | Window size, autoloads, renderer |
| A build/deploy pipeline | **Export presets** | `export_presets.cfg` → an Android APK |
| Unit/integration tests | Scripts run headless with `--script` | `check_*.gd` test scripts |
| A seeded random generator for reproducible jobs | `RandomNumberGenerator` with a seed | The course seed (chapter 6) |

## The game loop is your request loop

A web server waits for requests. A game never waits: it runs a **loop**, about 60 times a second:

1. Read input (touches, keys).
2. Update the world (`_process`, `_physics_process`).
3. Run physics (move bodies, detect overlaps).
4. Draw a frame.

Godot runs that loop for you. You write small functions that the loop calls:

```gdscript
func _physics_process(delta: float) -> void:
	position.y += Flow.speed * delta   # move down with the flood
```

`delta` is the time since the last call, in seconds (about 0.0167 at 60 fps). **Always multiply speeds by `delta`**: then a car moves 64 px per second whether the phone runs at 30 or 120 frames. This is the single most important habit in game code; it is like making a job idempotent: forget it and things break only on some machines.

- `_process(delta)` runs once per drawn frame: use it for visuals (animations, UI).
- `_physics_process(delta)` runs at a fixed rate (60 per second by default): use it for movement and game rules, so they behave the same on every device. Ulysses does almost all its logic here.

## Nodes and scenes: components you can nest

Everything in Godot is a **node**: a sprite, a physics body, a sound player, a button, a timer. Each node does one job. You build things by putting nodes inside nodes, like a component tree in React or an object graph in a service:

```
Boat (CharacterBody2D)        ← moves and collides
├── Hull (Sprite2D)           ← draws the boat
├── Rider (Sprite2D)          ← draws the person
├── Shape (CollisionPolygon2D)← the boat's solid outline
├── HullSensor (Area2D)       ← notices pickups and bumps
├── TopSensor (Area2D)        ← notices a car on the bow
└── BottomSensor (Area2D)     ← notices the bottom wall
```

Save a tree like this as a **scene** (`boat.tscn`) and you can drop copies of it anywhere, the way you instantiate a class. A scene can contain other scenes: `main.tscn` contains `boat.tscn` and `hud.tscn`.

A **script** attached to a node adds behaviour, like a class extending the node's type:

```gdscript
class_name Boat
extends CharacterBody2D
```

## Signals: pub/sub built in

A node announces that something happened with a **signal**; any other node can listen. This keeps parts decoupled, exactly like publishing an event instead of calling a service directly.

```gdscript
# boat.gd: the boat doesn't know what a "game over" is
signal sunk(reason: String)
...
sunk.emit("squeezed")

# game.gd: the game decides what sinking means
boat.sunk.connect(_on_boat_sunk)
```

Why it matters: the boat can be tested alone, and the game can change what happens on sinking (a crash sound, a sinking animation, a Continue offer) without touching the boat.

## Resources: data, not code

A **resource** is a data object saved as a file: a texture, a sound, an animation's frames, a UI theme, a shader material. Think of them as typed config files the engine loads and caches. Ulysses keeps:

- art and sound as imported resources (`.png`, `.ogg`, `.wav`),
- `SpriteFrames` (`assets/frames/*.tres`): which images make an animation,
- `Theme` (`scenes/ui/hud_theme.tres`): fonts and button styles for every menu,
- `ShaderMaterial` (`assets/shaders/*.tres`): which effects a sprite uses.

`preload("res://...")` loads a resource when the script loads (like an import); `load(...)` loads it when the line runs (like a lazy require).

## Autoloads: your singleton services

An **autoload** is a scene or script that Godot creates once at startup and keeps alive across scene changes, reachable by name from anywhere. Ulysses has one: `Audio` (`scripts/audio.gd`), so music keeps playing when you go from the menu to a run.

Use autoloads sparingly, as you would global singletons: for genuinely global services (audio, settings). Ulysses keeps game rules out of autoloads on purpose; static classes like `Tuning`, `Flow` and `SaveData` hold shared data without needing a node.

## res:// and user://

- `res://` is the project folder (read-only once exported): code, scenes, art.
- `user://` is a per-user writable folder (on Windows `%APPDATA%\Godot\app_userdata\Ulysses`, on Android the app's private storage): the save file lives here.

## The editor is your IDE and your admin panel

The Godot editor is where you build scenes visually, set node properties in the **Inspector**, and run the game (**F5**). Many things you might do in code (positions, sizes, which texture a sprite shows) are data in the scene file, set in the editor. Ulysses mixes both: layout lives in `.tscn` files, behaviour in `.gd` scripts, and numbers in `tuning.gd`.

## What carries over from backend work, and what doesn't

Carries over well:
- **Separation of concerns.** Each script has one job (the boat moves; the director spawns; the game scores).
- **Config in one place.** `tuning.gd` is the game's `.env`: every speed, chance and price.
- **Tests.** Headless test scripts check rules exactly (chapter 12).
- **Determinism.** A seeded generator makes a "random" run replayable, like replaying a request log.

Different in games:
- **Time is a variable.** Everything happens over frames; "after 2 seconds" is a timer, a tween or a counter, never `sleep`.
- **Feel matters as much as correctness.** A rule can be right and still feel bad. Chapter 5 shows how momentum, telegraphs and fair randomness make the same rules feel good.
- **The screen is the contract.** Tests check numbers; screenshots check that it looks right. Ulysses uses both.
