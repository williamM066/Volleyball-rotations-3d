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

**Set calls, and why tempo is the whole argument.** Pick 4, Go, 5, 1, 2, 3,
Slide or Pipe. Each says where the ball goes, who runs it, and how long the
other team has to do something about it. That last number is the point.

Their three blockers stand at the two pins and the middle, and they all have
to *travel* - at the same shuffle speed every other player obeys, less about a
quarter of a second to read the set. So the block that forms is derived, not
written down:

| Call | Tempo | What the other team manages |
|---|---|---|
| 4 (high outside) | 1.15 s | clean two-man block |
| Go (fast outside) | 0.78 s | second blocker stops 1.7 m short: **one pair of hands** |
| 1 (quick) | 0.42 s | **one blocker**, and their hands are still on the way up |
| 2 (high middle) | 1.00 s | closed double block - and it gets stuffed |
| 3 (shoot) | 0.65 s | two hands, but late |
| Slide / Pipe | ~0.7-1.0 s | closes, because the block is already on that side |

A blocker also needs about three quarters of a second to reach full
extension, so against a quick their hands only get to about 2.6 m and the ball
goes over the top. That the "2" gets blocked is not a bug: it is the tool
making its own argument for running tempo instead.

**Who covers what.** Toggle it on and the floor is tinted by who would play a
ball landing there - the same `whoTakes()` that answers when you click the
court, run over a grid. The seams you can see are the seams the tool would
actually name, and they move as you drag people. This is what turns a
defensive shape from six dots into an argument.

**The block is real.** A hitter does not swing at a fixed spot regardless of
who is in front of them. `aimShot()` casts rays from the contact point past
the block's outer hands: anything crossing the net inside those hands and
below them is blocked, so the hitter takes the widest corridor the block has
left, and the coaching note names which one. Consequences that fall out of the
geometry rather than being written down:

- Block set inside, taking the angle -> the hitter goes **line**.
- Block slid to the line -> the hitter goes **cross**.
- A wide or closed block, or a tight set where the contact is low -> **blocked**,
  and the ball comes straight back down inside three metres of the hitter,
  which is exactly where the cover cup is standing.

It is the same geometry the lit floor is drawn from, so what the tool shows
and what the ball does now agree. Before this, the ball flew through the
block: it crossed the net at x = -2.36 and 2.75 m, straight through hands
spanning [-3.23, -1.67] that reach 3.15 m.

**Four viewpoints, tweened not cut.** A formation is a different question from
each angle, so the camera snaps to the one that answers it. **Top** is how a
coach draws a rotation on paper, and the only view where spacing and seams are
honest. **Side** shows ball height, jump height and whether the block actually
got above the tape. **Behind** is the hitter's own view of what the block left
open. **Angle** is the general one. Dragging the court takes the camera off a
preset. Keys: `t` `s` `b` `a`.

**A ball trail.** A fading ribbon of where the ball has just been - the single
cheapest thing that makes a rally followable, because you can see what the
ball did rather than only where it is. It is read out of the cached
simulation, so it is exact at any scrub position instead of accumulating while
playing.

**Shareable links.** A picture is only useful to a coach if they can send it to
a player. The whole state - situation, rotation, back-row system, libero, both
markers, and anyone dragged out of position - packs into the URL. No account,
no server, no storage. "Copy a link to this picture" in the side panel.

**Movement is simulated, not interpolated.** The scenario says where it would
*like* everybody to be at a given moment; `intent()` reads that off the
keyframes. It does not move anybody. `simStep()` then moves real bodies toward
those targets under a locomotion model: acceleration of 8.5 m/s², a top speed
that depends on what they are doing (4.6 m/s running, 2.8 backpedalling, and
2.5 shuffling sideways along the net, because blockers do not cross over at
speed), and braking so they arrive rather than overshoot. Going for a ball is a
*target*, not a teleport: a defender runs at it and may still be short when it
lands.

Two consequences worth having:

- **You must be planted to jump.** Vertical is scaled by how settled you are,
  so a blocker who is late is caught mid-shuffle with their hands down instead
  of gliding into position already airborne. Measured across 86,616 samples:
  no blocker ever leaves the floor while travelling. The only players airborne
  while moving are hitters mid-swing, which is correct — a spike approach
  converts run into jump.
- **If the rally does not give them time, they do not arrive.** A block that
  cannot close is the most useful thing this tool can show.

Movement is stateful, which would normally break scrubbing. It does not here,
because the clip is re-simulated from zero at a fixed 1/60 s step and cached:
the same time always gives the same picture. A six second rally is about 360
frames, cheap enough to redo on every drag.

**Poses** — an attack is not a pose, it is a plant, a jump that starts only
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
- Anything inside a step's `opp` or `zones` map is treated as a person to
  interpolate. Putting metadata in there - the block returning its own `xs`
  and `joined` flags alongside its three players - takes the whole scene down
  on the first frame, because the interpolator reads `.p[0]` off a flag. The
  block now returns `{players, xs, joined, late}`, and the interpolator skips
  entries with no position rather than dying on them.
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
