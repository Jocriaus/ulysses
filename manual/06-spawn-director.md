# 6. The spawn director: fair procedural generation

`scripts/spawn_director.gd` decides everything that enters the flood. It is the most interesting system in the game, and the closest to backend work: it is an algorithm with invariants you can test.

## The problem it solved

The Unity prototype had eight spawn points, each with its own timer (5 to 9 s, fixed for the whole run) and a plain random pick. That produced:

- four cars side by side with no gap (an unfair death),
- rhythms (two points with the same timer always arrived together),
- an empty screen for the first 5 to 9 seconds,
- cars further apart as the flood sped up (timers are in seconds, the flood in pixels).

## Rows by distance, not time

```gdscript
func _physics_process(delta: float) -> void:
	_until_row -= Flow.speed * delta          # count pixels of water, not seconds
	while _until_row <= 0.0:
		var drifted := -_until_row             # how far past due we are
		_until_row += Tuning.ROW_SPACING       # 72 px
		spawn_row(Tuning.SPAWN_Y + drifted)
```

A row every 72 px of water keeps the flood equally crowded at any speed. The `while` and `drifted` keep rows exactly 72 px apart even when one frame covers more than a row's worth.

## The guaranteed safe path (the invariant)

Every row keeps a 40 px gap. Between rows the gap may move only as far as the **slowest boat** can steer in **half** the time the next row takes to arrive:

```gdscript
func max_shift() -> float:
	var slowest := Tuning.BOAT_SPEED * (1.0 + slowest_boat_speed_bonus)
	return Tuning.ROW_SPACING * slowest / Flow.speed * Tuning.REACH_FAIRNESS   # 72 px at 1x, 36 px at 2x
```

Items in a row may not touch the old gap, the new gap, or the space between them. So there is *always* a way through, for every boat, at every speed. This is the kind of guarantee you would assert in a backend test, and the test does: 5,000 rows, zero violations, and a bot steering the slowest boat through 60 s at normal speed and 40 s at double without a single bump.

**Alternatives:** generate rows freely and *check* reachability afterwards (path-finding, then re-roll); or hand-design every level. Constraining generation so the invariant holds by construction is cheaper and never fails.

## Shuffle bags: random that feels fair

A plain random pick can give the same car five times in a row. Players call that "unfair" even when it is "random". A **shuffle bag** draws every item once before refilling, like dealing from a deck:

```gdscript
class Bag:
	func draw(fits := Callable()):
		if _left.is_empty(): _left = _contents.duplicate()
		# never the item drawn last time, if anything else would do
		...
```

Bags decide car or trash (two of each per four draws), which car (each car twice, trees and posts joining after the first boss and growing more common), and which trash. A `fits` callback lets a wall ask "something no wider than 11 px?" without disturbing fairness.

## Patterns, waves and pickups

On top of random rows:

- **Patterns** (a bag of hand-made shapes): a wall with one gap, a funnel narrowing to the gap, a convoy (the same car four times in one lane), a zigzag. Harder ones unlock after bosses. Each pattern row still obeys the safe-path rule.
- **Waves of pacing**: 12 to 18 busy rows, then 3 or 4 rows of open water to breathe, and always calm water for the last 12 m before a boss.
- **Pickups placed with intent**: coins in trails of four along the gap (they show the way), people beside hazards off the path (a rescue is a choice), gas cans more likely as the tank empties (one promised soon after it drops below 25%), stopwatches, waves and rooftops in free spots.
- **Rare events**: a breather sometimes becomes a coin shower or a group of people, at least 80 rows apart.

## Seeds: reproducible randomness

```gdscript
seed_used = course_seed
if seed_used == 0 and Missions.daily_requested: seed_used = daily_seed()   # YYYYMMDD
if seed_used == 0: seed_used = seed_from_command_line()                   # --seed=N
if seed_used == 0: seed_used = randi() | 1
rng.seed = seed_used
print("Course seed: %d" % seed_used)
```

Everything the course is made of draws from this one `RandomNumberGenerator`; sounds and particles use Godot's global generator, so they never shift the course. The same seed gives the same course for the same play, which makes **the Daily** (everyone gets today's course) and **bug reports** ("seed 2774782527, the third wall") possible. This is the same idea as replaying a request log against a deterministic service.

## Performance

Placing a row (with everything in it) takes about 0.2 ms on a desktop, against a 16.7 ms frame. At one or two rows a second, nodes are instanced as needed. **Object pooling** (reusing nodes instead of creating and freeing them) is the usual alternative for bullet-heavy games; measured here, it would add complexity for no visible gain.
