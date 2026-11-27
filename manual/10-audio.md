# 10. Audio

## A global audio service

`scripts/audio.gd` is the project's one **autoload**: a node Godot creates at startup and keeps across scene changes. Any script plays a sound by name:

```gdscript
Audio.play("coin")
Audio.play_music("boss")
```

It owns two kinds of player:

- one **`AudioStreamPlayer`** for music, so switching tracks (menu → gameplay → boss) is one call, and the player itself survives scene changes;
- one `AudioStreamPlayer` per named effect, so different sounds (a coin and a hit) play at the same time, and a repeat of the same sound restarts it.

The node's `process_mode` is *Always*, so it keeps playing while the game is paused: the crash sound finishes over the frozen game-over screen instead of being cut off.

A table maps names to files, so game code never hard-codes a path:

```gdscript
const EFFECTS := {
	"hit": preload("res://assets/audio/hit.wav"),
	"coin": preload("res://assets/audio/Pick up coin.wav"),
	"wave": preload("res://assets/audio/wave.wav"),
	"low_fuel": preload("res://assets/audio/Select.wav"),
	...
}
```

Sound effects get a slight random pitch and volume each time, so repeated sounds don't feel mechanical. This uses Godot's global random generator, never the course seed, so sounds can't change a seeded course.

## Formats

- **Music** is OGG (compressed, streamed): the four tracks are about 3.6 MB together instead of 44 MB as WAV.
- **Short effects** stay WAV: decoded once, played with no delay.

## Settings and the app lifecycle

Music on/off and the two volumes are saved in `settings`. When the app goes to the background, the run pauses (`NOTIFICATION_APPLICATION_PAUSED`) and the music holds.

**Alternatives:** placing an `AudioStreamPlayer2D` in each scene (positional sound) suits big worlds; for a one-screen game a global service is simpler, and **audio buses** (Godot's mixer) would be the next step for separate music and effects processing.
