# 12. Testing and tooling

Games are often "tested by playing". Ulysses also has automated tests, written the way a backend developer would want: they check rules exactly, run headless in seconds, and never touch real data.

## A test is a script that drives the game

The tests are in [`tests/`](../../tests/README.md) at the repo root, outside the Godot project so they are never exported into the APK. Godot can run a script instead of the game: `godot --headless --path ulysses-godot --script tests/check_u9.gd`. The script extends `SceneTree` (the thing that runs the game loop), loads a real scene, sets things up, steps time, and checks results:

```gdscript
extends SceneTree

func _initialize() -> void:
	SaveData.path = "user://test_u9_save.cfg"        # never the real save
	main = load("res://scenes/main.tscn").instantiate()
	root.add_child(main)

func _physics_process(delta: float) -> bool:
	...
	_ok(absf(burned - 25.0) < 0.5, "half stick burns 25/s")
	...
	quit(1 if failures else 0)
	return false
```

`--fixed-fps 60` makes every run take exactly 1/60 s per step, so timing checks are exact and the test runs faster than real time.

## Running them

```
powershell -ExecutionPolicy Bypass -File tests\run_tests.ps1                    # all 19, about 4 minutes
powershell -ExecutionPolicy Bypass -File tests\run_tests.ps1 check_u9 check_ui  # some
powershell -ExecutionPolicy Bypass -File tests\run_tests.ps1 -Shots             # UI screenshots
```

Each prints `RESULT: PASS` or `RESULT: FAIL (n)` with the failed checks, and the runner ends with "19 of 19 passed" and a non-zero exit code on any failure, so it can run in CI. `tests/README.md` describes every test and how to write one.

## What the tests cover

| Test | Checks |
|---|---|
| `check_spawn_director` | The safe-path invariant over 5,000 rows, bag fairness, patterns, waves, pickups, gas by need, events, seeds, a bot steering through 100 s |
| `check_u9` | The current, burn by throttle, momentum timings, score |
| `check_u11` | Boat traits, stopwatch, waves, each boss's attack, ring reachability with real bullets, carried rescues |
| `check_u12` | Game-over reason, background saving, warnings, shake, settings, HUD, round looks, goals, missions, the Daily, the tutorial played through by a bot |
| `check_polish` | Button feel, polished portraits, start-screen figures animated from the original art (first frame identical to it), start scenery |
| `check_ui` | The UI pass: Missions, Customize's boats and a full game-over panel fit their frames; no text under 5 px and no button under 12 px on any panel |
| older checks | Movement, walls, fuel gauge, pickups, bosses, shop, customize, pause, settings layout, sounds, game-over freeze |

## Protecting real data: a sandbox

Twice during development a test wrote fake progress into the real save, once because an old test never set `SaveData.path`, once because a test failed to load and Godot ran the real game instead. The fix is the same one you'd use for a database: **tests get their own environment**. The test runner (`tests/run_tests.ps1`) points `APPDATA` (where Godot's `user://` lives) at `tests/.sandbox`, backs up the real save first, and restores it if anything changed.

## Screenshots: tests for the eye

Numbers can't tell you that a font is wrong or a sprite is clipped. A screenshot script runs the game in a window for a moment and saves the screen:

```gdscript
root.get_texture().get_image().save_png(path)
```

Every visual change in this project was checked this way. Screenshots caught a wrong font, a broken start screen, a too-large button, too-dark sprites and a clipped waving arm, none of which the number tests could see.

## Determinism

The course seed (chapter 6) makes the hardest system testable: the same seed gives the same rows, so a test can check "this exact course has no unfair row", and a bot can replay it. If you build anything random, give it its own seeded generator from day one.

## Tooling summary

| Tool | Used for |
|---|---|
| Godot editor | Scenes, the Inspector, running and debugging |
| Godot headless (`--headless`, `--script`, `--export-debug`) | Tests, importing, exports, in a terminal or CI |
| Aseprite (batch, Lua) | Drawing and editing art (chapter 9) |
| git | Scenes and scripts are text, so they diff and review like code |
| Python | Small edits across scene files during development |
