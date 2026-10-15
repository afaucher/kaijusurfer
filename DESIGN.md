# Kaiju Surfer: game design

**Status:** draft based on the playable prototype (`index.html`). Rules marked **(proposed)** are waiting on a decision; see [Open decisions](#13-open-decisions). Every number comes from the prototype and can be tuned. They're collected in the [tuning appendix](#appendix-tuning-values).

## 1. Pitch

You are a purple, many-armed kaiju crawling up through an endless city laid out as a maze. Pluck the power grid as you go, eat the soldiers sent to stop you, and blast through walls when the maze boxes you in. Keep climbing, because the whole army is coming up the roads behind you.

**Platform:** HTML5, mobile first, portrait, one-handed.

### Pillars

1. **Always moving.** You set a direction, not each step. The pressure comes from the wave, not from fiddly input.
2. **Route choice is the skill.** Every junction trades off safety, energy and speed. Roads with more energy charge you faster.
3. **See it coming.** Every big attack is telegraphed in red. Getting hit should feel fair.
4. **The monster is the health bar.** Damage shows up on the kaiju itself as lost arms, and losing arms changes how it plays.

## 2. Core loop

1. **Climb:** steer through the maze, always trending up the screen.
2. **Harvest:** the plucker arms grab pellets from the roadside as you pass. Crack open cells and cores to absorb bigger bursts.
3. **Spend:** use the laser to cut through walls and enemies. Use the claw on anything in reach.
4. **Survive:** dodge telegraphed strikes, eat soldiers to heal, and stay ahead of the wave.

A run ends when the kaiju dies. Score is blocks climbed.

## 3. Controls

| Input | Action |
|---|---|
| Tap the left or right half of the screen | Turn left or right at the next junction |
| Swipe up or down | Climb, or back off |
| **Claw** button | Short-range melee swipe in front |
| **Laser** button | Fire the laser in the direction you're facing |
| Desktop | Arrow keys or WASD to steer, Space for the laser, X for the claw, P to pause |

### Movement rules

- The kaiju keeps moving in its current direction until something changes it.
- **Hitting a wall while moving sideways:** turn up if possible, otherwise down, otherwise reverse.
- **Hitting a wall while moving up or down (proposed):** turn toward the side you last tapped, then the other side, then reverse.
- **Early taps** are remembered for the next 2 junctions.
- **Late taps** still count: within 0.3 of a block past a junction, the kaiju snaps back and takes the turn.
- Tapping the opposite direction reverses immediately, even between junctions.

The Claw and Laser buttons sit at the bottom center, between the left and right tap zones. The tap zones cover the rest of the screen.

## 4. The maze

- **5 vertical roads** across the screen, so there are **4 city blocks per row**. About 10 rows are visible on a typical phone.
- The maze is generated one row at a time as you climb. Each row is guaranteed to:
  - be reachable from the row below it, and
  - have at least one road up from every connected stretch.

  This means you can always make progress upward, but the way up may sit at the far end of a long sideways run.
- Roughly half of the sideways road segments are open, and a few extra roads lead up. Together these produce Pac-Man-style loops and the occasional dead end. The first 3 rows are fully open so the start is gentle.
- A wall is a road segment that's missing. **The laser can blast walls open permanently**, and the rubble stays visible.
- Rows far below the wave are thrown away.

## 5. City blocks and energy

Each block is a 3x3 grid of lots surrounded by sidewalks. Buildings and energy share the lots, so the city reads as a city and the energy reads as its power grid.

| Layer | Where | What | How you get it |
|---|---|---|---|
| **Outer** | Sidewalks facing an open road (4 per side, with some gaps) | Pellets | The plucker arms grab them automatically as you pass |
| **Middle** | The 8 lots around the center | 45% buildings, 55% energy cells. 60% of cells are full and 40% are empty | Break them open with the claw, the laser or any explosion |
| **Inner** | The center lot | 72% cores (70% of those are full), otherwise a tower | Break them open with the laser or an explosion. The claw can't reach them |

- **Breaking a full cell or core** releases an energy orb. If you're within about 2.6 blocks, the orb flies to you and is absorbed with a burst effect. Any orb not absorbed within **1 second** fades away.
- **Empty cells and cores** can still be broken, but release nothing. They look visibly dim.
- **Buildings** are cover and city texture. Explosions and the laser flatten them into rubble. They give no reward yet.
- Explosions don't take sides. Enemy strikes break energy nodes too, so a strike that misses you can leave energy lying around for you to grab.

## 6. The kaiju

- It moves at **2.6 blocks/s** at full strength. It's roughly one block wide, so its legs straddle the buildings along the road.
- It has **8 arms**:
  - **6 walkers**, which step in alternating tripod groups.
  - **2 pluckers** at the head, which reach about 0.6 block to snatch pellets and carry them to the mouth.
- **Health (proposed):**
  - The kaiju has 100 HP and loses one arm for every 12.5 HP lost. The severed arm flops away on the road.
  - Losing walkers slows you, down to 50% speed with no walkers left.
  - Losing pluckers cuts your harvest rate. With none left, you can only get energy from orbs.
  - Healing back past a threshold regrows an arm.
  - The kaiju dies at 0 HP.
- A non-continuous hit gives 0.3 s of invulnerability. Wave damage ignores this.

## 7. Weapons

### Claw (melee)

- Hits everything within **0.85 block** in front of the kaiju, in a wide arc about 200° across.
- 0.55 s cooldown, no energy cost.
- Eats marines and grenadiers (with the eating rewards), kills tanks outright, and cracks energy cells in reach. It never breaks cores.
- On screen, the pluckers whip outward and a crescent slash appears.

### Laser

- **Energy:** the meter holds 90 energy as 3 charges of 30. Each shot costs one charge, about ⅓ of the meter. **(Proposed:** you can fire whenever at least one charge is full.)
- **Beam:** reaches 5 blocks in the direction you're facing and blasts open **up to 2 walls**.
- **Blasted walls:** each one breaks the cells and cores right beside it, so a well-aimed shot pays back some of its cost.
- **Enemies:** the beam kills every enemy on its line.

## 8. Enemies

Enemies chase you Pac-Man style. They follow a distance map of the maze toward the kaiju, which is refreshed 5 times a second. They add a little randomness and avoid reversing unless they're stuck.

| Enemy | Moves | Attack | Telegraph | Counter |
|---|---|---|---|---|
| **Marines** (squad of 5) | Fast, with a lot of wander | 3-round rifle burst at close range, 1.3 damage per round | None. It's chip damage | **Edible:** +5 HP, +4 energy |
| **Grenadier** (alone) | Slow, keeps its distance | Lobs a grenade at where you're heading: 0.45 radius, 8 damage, also cracks energy nodes | Red landing ring while the grenade is in the air | **Edible:** +2.5 HP, +2 energy |
| **Tank** | Slow, holds its line | Shoots a shell straight down an open road, 9 damage | Flashing red aim line for 0.5 s | Claw or laser. Walking into it crushes the tank but costs you 6 HP |
| **Artillery** (off-screen) | — | 3 shells land inside a circle, 11 damage each | Big red circle with a shrinking countdown ring, 1.7 s | Leave the circle |
| **Airstrike** (off-screen) | — | A run of bombs along a road: a 5-block stretch of your column, or a whole row ahead of you. 13 damage each | Two parallel red lines with moving chevrons, 2.1 s | Turn off that road |

Every blast kills the soldiers it hits and breaks the energy nodes and buildings in range.

### Telegraph language

- **Red always means incoming damage.** Nothing else in the game uses saturated red.
- **Size matches threat.** A grenade gets a small ring, artillery a big circle, an airstrike a whole marked road.
- **The countdown is shown in shape**, through a shrinking ring or moving chevrons, so you can read it without a number.

## 9. The wave

- A dense band of tanks and infantry, with searchlights sweeping across it, fills the bottom of the screen and advances up the maze.
- **Speed:** 0.55 blocks/s at the start, rising to 1.5 blocks/s. If it falls more than 6 blocks behind, it speeds up to close the gap.
- **Contact (proposed):** about 26 HP per second while you're inside it.
- **Within 4.5 blocks:** it fires tracer rounds at you, 1.5 damage each.
- **Off-screen:** a "Wave N blk behind" chip appears so you always know how close it is.
- It feeds the maze, sending tanks, marines and grenadiers up the roads ahead of it.

## 10. Difficulty and spawning

Difficulty ramps from 0 to 1 over the first **150 blocks** climbed.

| Item | Start | Full difficulty |
|---|---|---|
| Wave speed | 0.55 blocks/s | 1.5 blocks/s |
| Spawns from the wave (50% tank, 40% marines, 10% grenadier) | every 3.4 s | every 1.5 s |
| Spawns ahead, at the top of the screen (mostly marines) | every 4.5 s | every 2.2 s |
| Artillery (starts after 6 blocks) | every 8 s | every 4.5 s |
| Airstrikes (start after 14 blocks) | every 14 s | every 8 s |
| Enemy speed | ×1.0 | ×1.35 |

No more than 13 enemies are on the map at once.

## 11. Run end and scoring

- The kaiju dies, explodes, and sheds its remaining arms. Then the summary screen appears.
- **Summary:** blocks climbed (the score), energy harvested, soldiers eaten and arms lost.
- **Later:** a best-score record, daily seeded mazes, and districts with their own palettes and bosses.

## 12. Presentation

### Visual style

There are two styles to compare, and both are live in the prototype.

- **Neon arcade:** Pac-Man-like neon outlines around the roads on a dark night city. Energy glows, enemies are warm colors, and the kaiju is purple.
- **Pixel city:** a low-res buffer drawn about 2 screen pixels per game pixel. Asphalt roads with lane markings, lit windows on the buildings between blocks, and sprites with hard outlines.

**Recommendation:** build on **Neon** for readability, and borrow Pixel's street cues (lane markings, lit windows) to keep the city feel.

### Color roles

| Role | Neon | Pixel |
|---|---|---|
| Kaiju | Purple `#c45bff` | Purple `#8e3fd8` |
| Energy | Yellow pellets, green cells, gold cores | Same family |
| Enemies | Orange (tanks), pink-red (infantry) | Olive and khaki (military) |
| Danger | Red `#ff1f45`, used only for telegraphs and the wave | Red `#ff3044` |
| Maze | Cyan road edges | Grey asphalt and sidewalk |

### HUD

- **Top left:** blocks climbed.
- **Top center:** the arms (8 pips, with the pluckers shaped differently) and an HP bar.
- **Top right:** the style toggle and pause.
- **Bottom:** the Claw button (with its cooldown sweep), the Laser button (with a 3-segment energy ring and the charge count), and hints for the tap zones.

### Feedback

- **Camera:** screen shake on hits, blasts and lost arms. It's turned off when the device asks for reduced motion.
- **Pop-up text:** "CHOMP", "+12", "-ARM", "ARM REGROWN".
- **Kaiju:** a white flash when it's hit, and a glow pulse when it absorbs energy.

### Audio

Not started. Each telegraph type should have its own warning sound (rising for artillery, jet whine for airstrikes) so you can react without looking.

## 13. Open decisions

The prototype uses the recommended option for each.

| # | Decision | Recommended | Alternative |
|---|---|---|---|
| 1 | When can the laser fire? | With at least one charge full (⅓ of the meter) | Only when the meter is full, then up to 3 shots |
| 2 | How do you climb on purpose? | Tap left or right, and swipe up or down. Walls push you upward | Always auto-turn up at the first open junction (removes route choice) |
| 3 | What does the wave do on contact? | Heavy damage over time | Instant death |
| 4 | Can the claw break cores? | No: cells only | Yes, but then the laser becomes optional |
| 5 | How do health and arms relate? | 100 HP, one arm per 12.5 HP, arms regrow | One arm per hit, 8 hits total |
| 6 | Which visual style? | Neon, plus Pixel's street details | Pixel city |
| 7 | Endless run or levels? | Endless, with districts later | Short levels with an exit at the top |

## 14. Next steps

1. Settle the open decisions above.
2. Play-test on real phones: turn-timing tolerances, button placement, frame rate.
3. Move to TypeScript and Vite, and split the code into modules (maze, movers, AI, weapons, render). Hold off on an engine unless real devices show we need one.
4. Add audio and haptics (vibrate on hits and lost arms).
5. Fixed block layouts (substation, residential, park) so routes are easier to read.
6. A best-score record, then districts and bosses.

## Appendix: tuning values

| Value | Setting |
|---|---|
| Road half-width | 0.17 block |
| Kaiju speed | 2.6 blocks/s × (0.5 + 0.5 × walkers left ÷ 6) |
| Pluck reach | 0.6 block |
| Pluck time | 0.075 s reach + 0.09 s carry |
| Energy | Pellet 0.6, cell 5, core 12. Meter maximum 90, one charge = 30 |
| Orb attraction range | 2.6 blocks |
| Orb lifetime | 1 s |
| Claw | 0.85 block range, 0.55 s cooldown |
| Laser | 5 blocks long, up to 2 walls. Beam hits enemies within 0.38 block. Wall blast breaks nodes within 0.56 block |
| Turn buffer | Early taps last 2 junctions. Late taps count within 0.3 block |
| Invulnerability after a hit | 0.3 s |
| Maze generation | 52% of sideways segments open. 28% chance of an extra road up. First 3 rows fully open |
| Block contents | Buildings 45%. Cells 60% full. Center lot is a core 72% of the time, and 70% of cores are full. 12% of pellet spots are left empty |
| Enemy speeds | Tank 1.25, marines 1.7, grenadier 1.25 blocks/s |
| Tank | 5-block range, 0.5 s aim, shell at 6 blocks/s, 2.6 s reload |
| Marines | 2.4-block range, 3-round burst, 1.1 s reload |
| Grenadier | Throws from 0.9 to 3.2 blocks away and backs off inside 1.6. 0.95 s flight, 3.2 s reload |
