# RATTLETRAP

**Build it, then live with it.**

Weld a machine together out of scrap in the yard, then take it out onto the wasteland and race it against the clock. The track will tell you what you got wrong.

▶ **Play: [rattletrap.vercel.app](https://rattletrap.vercel.app)**

---

## The idea

Every part you bolt on changes how the thing drives — and not because a "handling" stat went up or down. There isn't one. The physics simply cares where you put the weight.

Cannon's `RaycastVehicle` derives each wheel's grip from `frictionSlip × suspensionForce`, and suspension force is that wheel's share of the load. So weight distribution drives grip distribution on its own. Two things had to be honest for that to hold:

1. **The body origin sits at the true centre of mass**, because cannon assumes it does. Shape offsets are `blockPos − CoM`.
2. **The inertia tensor is computed by parallel-axis** and written over `body.inertia` / `invInertia`. Cannon otherwise approximates inertia from the bounding box, and a long chassis would yaw like a cube.

Aero drag is added by hand at the centre of mass — `RaycastVehicle` models none, so without it every build accelerates forever and the straight can't test anything.

That's the whole trick. Nothing about handling is authored.

## What that buys you

Four preset machines, one autopilot, one lap each:

| Build | Lap | |
|---|---|---|
| **THE RAIL** | 34.2s | Long, low, wide. Never puts a wheel wrong. |
| **THE TOWER** | 35.2s | Tall and narrow. Survives only because the bot drives it carefully. |
| **THE BRICK** | 37.9s | Balanced, but ran wide — 98 frames in the dirt. |
| **THE ANVIL** | 41.2s | 636 kg with ballast in the nose. Understeers into everything. |

Give it a 1-metre wheelbase and enough engine and it will backflip off the start line, because the rearward load transfer exceeds the static front load. That's not a special case anyone wrote. It's just true.

## The wasteland has opinions

- **Everything is solid.** Edge stakes are light dynamic bodies — clip one and it cartwheels away and costs you speed, rather than acting as a wall that deletes your run. Burnt-out husks, dead trees and shacks do not move. Neither do the overpass pillars.
- **Off-road costs you ~33%.** Dirt is modelled per-wheel: whichever tyres are off the road get rolling resistance and reduced grip, so putting two wheels in the sand drags *and* pulls you sideways. Cutting a corner is a real decision.
- **Roll it and it recovers.** On your roof and stopped for two seconds and it puts you back on the last checkpoint.

## The race

**Three laps, four machines.** The opposition is built out of the same parts you are and driven by the same physics — no grip bonus, no rubber-banding, no cheating. They just follow their own line and brake for corners. They're quick: a typical finish is decided by a few car lengths.

## The garage

Build it, name it, keep it. Saved machines live in your browser and load back with a click. Same name overwrites rather than duplicating.

## THE OLD 47

850 m, six stretches — each interrogating one decision:

- **THE STRAIGHT** — power against weight
- **THE CHICANE** — grip and wheelbase
- **THE WASHBOARD** — short wheelbases bounce
- **THE RAMP** — nose-heavy lands nose-first
- **THE OFF-CAMBER** — tall builds roll
- **MERCY** — what's left of the town the road was built for. The buildings crowd the verge and the water tower stands over it. Everywhere else is open plain; this is where you thread.

## Drift

Hold `Shift` into a corner and the machine locks into a slide. Steer within it — into the drift to tighten, out to run wide. Hold it and the sparks go amber, then red, then white-blue. Let go on white-blue and it fires.

The car really does travel at an angle to its nose; it isn't a mesh trick. Getting there took five attempts, and the answer was one line: **cannon's friction solver runs inside `world.step()` and re-aligns the velocity to the nose every frame**, so everything applied before the step was erased before it rendered. The velocity is steered *after* the step, with the tyres loosened so they don't fight it. Rotating a vector can't change its length, so a drift costs no speed — which is exactly the point of one.

Your build still decides the payout. The boost is an impulse and a force, so `F = ma` gives the 392 kg BRICK **+36 km/h** off an ULTRA and the 636 kg ANVIL **+21**. Same spark, different machine.

## Controls

| | |
|---|---|
| **Garage** | Click place · Right-click remove · Drag orbit · Scroll zoom · `1`–`4` pick part |
| **Driving** | `W` throttle · `S` brake / reverse · `A`/`D` steer · **`Shift` drift** · `R` respawn |

`Shift` + a direction locks a drift. Hold it to charge — amber, red, white-blue — and let go for the mini-turbo. (`Space` works too, if that's where your thumb goes.)

Wheels ahead of the balance point steer. They're the ones tinted blue.

## Sound

Fully synthesised — no files to license, host or wait on. A fake 4-speed gearbox, because the pitch rising and dropping on the shift is most of what makes an engine read as an engine. Plus tyre skid tied to actual slip, gravel when you're off the road, wind on speed, and impacts scaled by contact force.

## Running it

No build step, no dependencies to install — one HTML file, Three.js and cannon-es over CDN.

```
python -m http.server 5771
```

## Deploying

The Vercel project is connected to this repo: **pushing to `main` deploys to production.** No CLI step needed.

## Debug handle

`window.__dbg` is live in the console:

```js
__dbg.preset('THE ANVIL')   // load a build
__dbg.go()                  // start a run
__dbg.stats()               // mass, CoM, inertia, balance, frontal area
__dbg.auto(600)             // headless track-follower; drives real laps
__dbg.sim(120, ['w','a'])   // step physics with keys held (immune to rAF throttling)
__dbg.state()               // speed, progress, slip, off-road, wheels in contact
__dbg.look([x,y,z],[x,y,z]) // free camera; returns a JPEG data URL
```

`__dbg.auto()` is how those lap times were measured, and it's the seed for AI opponents.

---

Built with [Claude Code](https://claude.com/claude-code).
