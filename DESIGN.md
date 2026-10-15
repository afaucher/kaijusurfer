# Kaiju Surfer: design decisions

Status: prototype (`index.html`). Each item below is a place where the brief was ambiguous or two parts of it conflicted. Each one gives a recommendation and how the prototype handles it today. Items marked **Decide** need a call before production work starts.

## Core loop

| # | Question | Recommendation (what the prototype does) | Alternatives |
|---|---|---|---|
| 1 | What does the "5xN grid" mean? | 5 vertical roads across the screen, so 4 city blocks per row. The maze runs upward forever and is generated row by row. Every row is guaranteed at least one way up, and every node can be reached. | 5 blocks across (6 roads). That makes everything about 17% smaller on a phone. |
| 2 | Is the run endless, or split into levels? | Endless, and the score is blocks climbed. Difficulty ramps up over the first ~150 blocks. | Districts every ~50 blocks with a boss or a palette change. Add this later. |
| 3 | **Decide:** what does the wave do on contact? | Heavy damage over time (~26 HP/s), not instant death. A rubber band keeps it within ~6 blocks, so dawdling is always punished. Its speed ramps from 0.55 to 1.5 blocks/s; the kaiju moves at 2.6. | Instant death is cleaner but harsh once the maze dead-ends you. |

## Controls

| # | Question | Recommendation | Alternatives |
|---|---|---|---|
| 4 | **Decide:** taps only give left and right. How do you go up or down on purpose? | Tap the left or right half of the screen to turn. Swipe up or down to climb or back off. Walls redirect you automatically, so most climbing happens without input. | Up-only auto-climb: the kaiju always turns up at the first open junction. This is simpler, but it removes route choice. |
| 5 | The wall rule ("up first, then down") only covers horizontal movement. What happens when you hit a wall going up? | Turn toward the side you last tapped, then the other side, then reverse. | Always turn toward the side with more pellets. That feels smart but is less predictable. |
| 6 | How much turn tolerance at T-junctions? | Early taps are buffered for 2 junctions. A late tap within 0.3 block of a junction snaps back and takes the turn. | Tune these numbers on real devices. |
| 7 | **Decide:** the laser fires "when full of energy" but costs 1/3. Which rule wins? | The meter holds 3 charges, and you can fire whenever at least one is full. The button's ring shows the 3 segments. | Literal reading: the laser arms only at a full meter, then allows up to 3 shots. More "ultimate"-like, but slower. |
| 8 | How do you trigger the claw (melee)? | A dedicated Claw button next to Laser. It has a 0.55 s cooldown and costs no energy. | Double-tap anywhere. This conflicts with fast side taps, so it's not recommended. |

## The city and its energy

| # | Question | Recommendation | Alternatives |
|---|---|---|---|
| 9 | What are the 3 layers, now that buildings are mixed in? | Each block is a 3x3 grid of lots. **Outer:** pellets on the sidewalks, plucked as you pass. **Middle:** the 8 ring lots, each a building (45%) or an energy cell (full 60% / empty 40%). **Inner:** the center lot, a core (72%) or a tower. | Fixed layouts per block type (substation, residential, park) to make route reading easier. Worth trying next. |
| 10 | **Decide:** what breaks what? | Pellets are plucked only. The claw cracks cells but never cores. The laser, grenades, artillery and airstrikes break cells, cores and buildings. Energy from a broken full node flies to you if you're within ~2.6 blocks, and fades after 1 s. | Letting the claw break cores makes the laser feel optional. |
| 11 | What does the laser actually hit? | It reaches 5 blocks along your facing and blasts open up to 2 walls. It kills every enemy on that line. Each blasted wall also breaks the cells and cores right beside it, so a well-aimed shot refunds some of its cost. | Unlimited walls make the maze moot. |
| 12 | Can buildings be destroyed? | Yes, by blasts and the laser. They leave rubble and no reward for now. | Let some towers drop energy or score. |

## Health and the kaiju

| # | Question | Recommendation | Alternatives |
|---|---|---|---|
| 13 | **Decide:** how do health and arms relate? | 100 HP, with one arm lost per 12.5 HP (8 arms: 6 walkers, 2 pluckers). Losing walkers slows you. Losing pluckers cuts your harvest rate. Healing back over a threshold regrows an arm. | Arms as discrete hit points (8 hits). Simpler, but grenade chip damage and wave damage become awkward. |
| 14 | What happens when you eat soldiers? | Marine squads give +5 HP and +4 energy. Grenadiers give +2.5 HP and +2 energy. This is the main way to regrow arms. | |
| 15 | Can you eat tanks? | No. Walking into one crushes it but costs you 6 HP. The claw kills it cleanly. | |

## Enemies

| # | Question | Recommendation |
|---|---|---|
| 16 | How do enemies chase? | Pac-Man style: they follow a breadth-first-search distance map of the maze toward you, with some jitter, and avoid reversing. Marines wander more. Grenadiers keep their distance. |
| 17 | How are attacks telegraphed? | Tank: 0.5 s red aim line, then a straight shell. Grenade: a red landing ring while the grenade arcs. Artillery: a big red circle with a shrinking countdown ring, then 3 shells. Airstrike: two parallel red lines with moving chevrons along a road, then a run of bombs. |
| 18 | Is there friendly fire? | Yes. Strikes kill soldiers and break energy nodes. That makes strikes a risk-and-reward moment: dodge the run, then grab the energy it releases. |

## Visual style

**Recommendation: build on Neon arcade, and borrow Pixel city's street cues.** Neon's glow keeps 2-4 px pellets and the red telegraphs readable on a small, bright phone screen. It's also vector, so it stays sharp at any pixel density and needs no sprite pipeline yet. Pixel city reads more like a *city* (lane dashes, lit windows, rooftops). But at mobile resolution, pellets and cells shrink to 1-2 px. Both styles are live in the prototype; the toggle is in the HUD and on the title screen.

## Tech

- Vanilla Canvas 2D in one file, no dependencies. Both styles share one set of draw calls with two rasterizers: vector plus glow sprites, and a hand-snapped low-res buffer.
- Recommended next step: move to TypeScript and Vite and split the code into modules (maze, movers, AI, render). Hold off on an engine until real-device performance says we need one.
- Not yet tested on physical devices, and there's no audio yet.
