# Volleyball Court Vision

A 3D volleyball court for coaching rotations, defence and transition.

**[Open it here](https://williamM066.github.io/Volleyball-rotations-3d/)**

## What it does

- **Three situations** — serve receive, their attack, a free ball. Each one runs
  all the way through: first ball, set, swing, and where it lands.
- **Drag anything.** Two coloured markers set up the rally; drag any player and
  everything downstream follows — who passes, who sets, who finishes, and
  whether the ball gets blocked.
- **Rotations R1–R6**, 5-1, libero on or off, perimeter or middle-up defence.
- **Set calls** — 4, Go, 5, 1, 2, 3, Slide, Pipe — each with its real tempo, so
  a quick beats the block across and a high ball does not.
- **A real block.** Every blocker in the air is a pair of hands at a height you
  set. Put somebody in front of the hitter and the ball stops; take them away
  and it goes through.
- **Area to defend** — the floor a hard swing can still reach, lit up.
- **Who covers what** — the court tinted by whose ball it is.
- **Squad tab** — add or remove players on either side.
- **Plays tab** — build your own sequence of touches and save it.
- **Share a link** — the whole picture packs into the URL. No account, no server.

## Controls

| | |
|---|---|
| Drag a player or marker | Move them |
| Drag bare court | Orbit the camera |
| Click bare floor | Who takes a ball there |
| `1`–`6` | Rotation |
| `t` `s` `b` `a` | Top, side, behind, angle view |
| `Space` | Play / pause |

## Run it locally

```bash
git clone https://github.com/williamM066/Volleyball-rotations-3d.git
cd Volleyball-rotations-3d
python -m http.server 8000
```

Then open `http://localhost:8000`. It has to be served rather than opened as a
file, because the page loads Three.js as an ES module.

## How it works

One file, `index.html`. Three.js from a CDN, no build step and no dependencies.
Players are articulated figures posed by joint angles; the rally is generated
from the markers rather than animated by hand, and movement is simulated at a
speed a human can manage — so a blocker who does not have time to get there
simply does not arrive.
