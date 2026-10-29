# Volleyball Court Vision

A 3D volleyball court that shows where each player should be as the ball moves.
Built around a 5-1 system, aimed at helping a player understand their own job.

Single self-contained `index.html`. No build step, no dependencies to install —
Three.js loads from a CDN at runtime.

## Run it locally

Open `index.html` in a browser. That's it.

(If your browser blocks ES module imports from `file://`, serve the folder
instead: `python -m http.server 8000`, then open <http://localhost:8000>.)

## Publish on GitHub Pages

Pages is free for **public** repositories. If the repo is private you need a
paid plan, so make it public unless you have a reason not to.

```bash
git init
git add index.html README.md
git commit -m "Volleyball court vision"
git branch -M main
git remote add origin https://github.com/<you>/volleyball-court-vision.git
git push -u origin main
```

Then on GitHub: **Settings → Pages → Source: Deploy from a branch →
`main` / `/ (root)` → Save.**

Your link appears in a minute or two at:

```
https://<you>.github.io/volleyball-court-vision/
```

To use the bare `https://<you>.github.io/` address instead, name the repository
exactly `<you>.github.io` and push the same files to it.

## What's in it

**Three situations, one rally.** Serve receive, their attack, and a free ball.
Each starts differently and then runs the same way through to the end: our team
plays the first ball, the setter releases to the target, a hitter finishes it,
and the ball lands on their floor. Nothing stops at the dig.

**Two markers drive everything.** Each situation has a pair of draggable
markers — the server and where the serve lands, their setter and where the set
goes, or where a free ball comes from and lands. None of the rally is a
hand-authored clip: `buildRally()` generates the whole thing from those two
points, so moving a marker changes where the first contact happens, who makes
it, who is then allowed to set (a setter who dug it cannot), which pin finishes
the attack, and where the opposing block forms.

**Drag anybody** — grab any player, including the opponents, and put them
somewhere else. The rest of the simulation follows rather than ignoring you:

- The lit floor is rebuilt from where the blockers and the hitter actually are.
  It is pinned to the geometry of the step that cast it, not to positions
  still sliding toward the next step, so it holds still while the blockers
  land and the hitter walks away; when the rally moves past it, it fades out
  instead of swelling and sliding across the court.
- The ball is anchored to people, not to coordinates, so a setter dragged into
  the back row sets from the back row.
- Who plays each ball is decided from live positions, so dragging someone
  closer makes them the one who digs it.

"Put N back" in the side panel undoes it. Changing rotation, scenario or the
libero also resets it.

**Nobody stands inside anybody else** — a relaxation pass keeps players at
least 1.15 m apart. A player you placed by hand is treated as immovable, so the
other one gives way.

**Rotations** — all six rotations of a 5-1, keys `1`–`6`. Roles are computed
from the rotation rather than placed by hand.

**Libero toggle** — on, the libero replaces whichever middle is in the back row.
Off, that middle stays in and has to pass. Serve receive shows the difference.

**The switch** — rotational order only has to hold at the moment the server
contacts the ball. After that, players move to the job they actually play: the
middle blocker to the middle of the net, the outside to the left pin, and in the
back row the middle blocker (or the libero covering for them) to left back. So a
middle who served from position 1 does not spend the rally stranded in the right
back corner. Serve receive shows this directly: the three passers hold a legal
rotational shape until the ball is struck, then the whole team moves.

**Follow one player** — picks a *person*, not a court position. Change rotation
and you keep watching the same player; the label tells you which zone they are
in now. If the selected middle is off the court in that rotation, it follows the
libero covering for them and says so.

**Your area** — with a player selected, the floor they are responsible for
glows. It is computed, not drawn: a grid is swept over the court and every cell
is asked the same "who takes this ball" question, so the patch always matches
the positions actually on screen. Drag a team-mate and the patch reshapes.
Blockers get no patch, because their job is the ball at the net.

**Double contact** — nobody may touch the ball twice in a row. The setter ends
up digging more often than people expect, especially at right back, and when
that happens the right side takes the second ball instead. `setterAfter()`
handles it and the panel says what is going on.

**Movement** — an attack is not a pose, it is a plant, a jump that starts only
once the feet have loaded, the arm drawn back, the contact and the
follow-through. A block is the same shape: hands up on the floor first, then
the jump, then landing once the ball is past. Scenarios only say "approach"
then "swing"; `motion()` derives the rest from the keyframe either side and
the ball height, so every scenario gets the movement without authoring it.
Bodies ease between poses rather than snapping — except on a scrub or a
scenario change, where there is no movement to show and they snap instead.

**Area to defend** — the lit floor is computed, not drawn: rays from the
attacker's contact point past the blockers' outer edges, projected down to the
court, and drawn back to that contact point so it reads as something coming out
of the smash. Whatever the block covers goes dark; what stays lit is every spot
a hard swing can still reach, which is exactly where the back row has to be
standing. It reaches forward into the 3 m zone, because the contact point is
modelled close to the net.

Three constants drive it, at the top of the script and in each defensive
scenario: `NET_H` (2.43 m), `HAND_H` (how high the blockers' hands reach over
the net, 3.15 m), and the attacker's contact point `A = [x, y, height]`. Lower
`HAND_H` or raise the contact and more of the deep court lights up, because the
ball starts clearing the block. It assumes a closed block and a straight swing,
so treat it as the geometry of a well-formed block rather than a prediction.

**Defence to offence** — the transition sequence, which is where most rallies
are actually won. The footwork follows standard coaching guidance: land on two
feet off the block, get off the net *before* getting wide, and turn and run
rather than back-pedal. The outside finishes outside the antenna, since left
side attackers take well over half of all transition swings.

**Back-row system** — Perimeter vs. Middle up. Put the set marker on a pin in
"Their attack", then toggle the system and watch who ends up with the dig.

**Landing cross** — marks where the ball is about to come down while it is still
in the air.

**Click the court** — anywhere on the floor, to get who takes that ball, how far
they have to travel, and whether the block already covers that spot.

## Changing the volleyball

The defensive positions are the opinionated part and are meant to be edited.
They live in two functions in `index.html`:

- `defRight(system)` — positions against an attack on our right side.
  `mirrorSetup()` derives the left-side version automatically, so only edit this one.
- `defMiddle(system)` — positions against an opponent middle quick.

These are keyed by **defensive slot**, not by rotational position. `switchMap()`
decides which player ends up in which slot:

```js
const FRONT_SLOT = {MB:'P3', OH:'P4', OTHER:'P2'};   // middle, left pin, right pin
const BACK_SLOT  = {MB:'P5', OH:'P6', OTHER:'P1'};   // left back, middle back, right back
```

`OTHER` is the setter or the opposite, whichever is in that row. If your libero
plays middle back in defence rather than left back, swap the `MB` and `OH`
entries in `BACK_SLOT`.

Each entry is a court coordinate in metres plus the coaching note shown in the
side panel:

```js
P6: rot
  ? {p:[1.9,3.7], pose:'ready', tag:'Middle up', note:'Step up behind the block...'}
  : {p:[0.3,8.0], pose:'ready', tag:'Deep middle', note:'Stay deep...'}
```

Coordinates are metres: `x` runs sideline to sideline (`-4.5` to `4.5`), `y` runs
from the net (`0`) to the endline (`9`). The attack line is at `y = 3`.

## Known simplifications

- Serve-receive formations are generic rather than per-rotation, and do not
  enforce overlap rules.
- Players are articulated 3D bodies built from capsules: hips, spine, head,
  two arms and two legs, each a real joint. A pose is a set of joint angles
  (`POSES3`), and the whole figure turns to face the ball, so a dig platform
  points at the ball from any camera angle with no mirroring tricks. The
  contact point is read off the actual hand bones with `getWorldPosition`, so
  the ball genuinely arrives in the hands.
- Two sign conventions are easy to get wrong if you edit `POSES3`. A limb
  hangs down at 0, and `rotation.x` swings it **forward** (`PI` points it
  straight up). The spine is the opposite: its offset points up, so a positive
  `chest` rotation leans the body **backward** and forward lean is negative.
- A defender covers about two metres to a ball already on its way down. If
  the nearest player is further than that when it lands, the panel says
  "Nobody reaches this — it drops" instead of naming a digger who never
  touched it. That is what makes the perimeter-versus-middle-up comparison
  honest: in Tip over the block, perimeter now genuinely concedes the tip and
  middle-up genuinely digs it, from the geometry rather than from the caption.
- The ball is a real volleyball: three sine bands a third of a turn apart on
  an equirectangular canvas, which become the classic wrapped panels once they
  are on a sphere, with seams along the band edges. It spins in proportion to
  how far it moves.
- Each end of a ball flight is anchored to where that player stands AT THAT
  STEP, not to where they are right now. Using the live position for both ends
  meant the ball's own start and finish slid around while it was still in the
  air, which is what made flight look broken.
- Responsibility is sticky. Two players who are nearly tied used to swap it
  frame to frame, and since the ball snaps to whoever holds it, it flickered
  between two pairs of hands.
- Serve receive satisfies the overlap rules in all six rotations, with the
  libero on and off. Position is judged by where the feet are when the server
  contacts the ball: each front-row player must be nearer the net than their
  back-row counterpart, and left-to-right order must hold within each row.
  `legalise()` nudges an authored shape until all seven constraints hold,
  which beats hand-placing six players legally in twelve different cases.
- Players stand in their ROTATIONAL spot at the serve, not their switched one.
  The switch only happens once the ball has been struck. Using the switched
  spot at the serve put the opposite on the wrong side of the middle, which is
  an overlap fault, and made a front-row player look like a fourth back-row
  passer.
- The contact pull on the ball eases off over about a third of a second after
  a contact. Releasing it in a single frame, which is what happened when
  responsibility switched off, made the ball jump back onto its path - the
  stutter you see when the setter plays it.
- A front-row middle runs their approach even when the set is going outside.
  Standing and watching lets the opposing middle blocker leave early and
  double the pin, so a decoy quick is part of the attack, not decoration.
- A serve contact is instant; the server drops out of the serve pose within a
  fraction of a second and follows the ball in, rather than holding the pose
  for the whole flight and then teleporting onto the court.
- Cup and cover spots are handed out in the order players already stand in,
  left to right, not by zone number. Handing them out by zone number made two
  players swap ends of the court on the way into the cup, crossing straight
  through each other at nearly 6 m/s - which reads as teleporting, not as
  volleyball.
- The setter releases to the target during the serve, and a hitter who is not
  playing the first ball starts transitioning while the ball is still in the
  air. Both of those are what real players do, and both exist because without
  them the same movement had to happen in one second afterwards. Worst
  movement anywhere is now about 3.4 m/s, down from 5.6.
- Responsibility for a ball only applies while it is below 1.6 m. Without that
  gate a passer stayed responsible for the whole flight of their own pass and
  got dragged across the court after it.
- x runs the same way on both sides of the net, so a blocker facing a hitter
  stands at roughly the SAME x, not the mirrored one. Mirroring it puts the
  block on the wrong side of the court, which is easy to miss on screen.
- Blockers stand 0.9 m apart, not 1.35 m. The wider spacing left exactly the
  seam the coaching note warns about.
- Defenders step in up to 0.45 m when the ball is coming down in front of
  where they stand. It is deliberately small: it should read as adjusting to
  the ball, not as abandoning the position the scenario is teaching.
- Running between positions is not animated. Players ease between poses and
  slide between spots; there is no gait.
- A player reaches at most 1.6 m for a ball. Anything further away drops, which
  is deliberate: it is what makes the perimeter-versus-middle-up comparison
  honest.
- The opponent is represented by up to three figures, not a full six.
