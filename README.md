
# Particle Foam — A Living Field of Light

*A generative visual experiment in motion, color, rhythm, and emergence.*

https://beauterre.github.io/foam/

Thousands of tiny colored particles drift across a full-screen canvas, forming a constantly evolving field of luminous cells. Colors gather, blend, shimmer, and dissolve. Fleeting structures emerge from apparent randomness, hold together for a moment, and then disperse into new formations.

The result feels like **a massage for the eyes**: a gentle, hypnotic visual rhythm somewhere between glowing foam, bioluminescent water, a field of distant stars, and a living microscopic world.

There is no predetermined animation, no fixed pattern, and no final state. Everything emerges from a few simple rules applied repeatedly.

## The Experience

The screen is alive with thousands of small lights. Most are pale, luminous colors; a smaller proportion are dark, creating contrast and depth.

Beneath the individual particles, a coarse grid acts as a second visual layer. Each cell takes on the average color of the particles inside it, gradually blending toward new colors as the particles move between cells.

The result combines three visual rhythms:

- **Fast:** individual points and overlapping glows create a glittering texture.
- **Medium:** particles drift, gather, and disperse, producing moving clusters.
- **Slow:** broad regions of color transform, creating patterns that appear and disappear like foam.

These layers interact to produce an effect that feels both restless and soothing. The eye can follow individual sparks, but it can also relax into the larger movement of the field.

## How It Works

### 1. Eight Thousand Independent Particles

The simulation begins with 8,000 particles randomly distributed across the canvas.

Each particle has:

- An RGB color.
- A position (`x`, `y`).
- A velocity (`dx`, `dy`).
- An individual blinking parameter.
- A random phase that determines its starting point in the blinking cycle.

Approximately 70% of the particles receive relatively bright colors, while the remaining 30% receive darker colors. This imbalance creates a luminous field with scattered dark accents rather than a uniformly bright cloud.

Initially, every particle is stationary. All movement emerges from the forces applied during subsequent animation frames.

### 2. A Grid That Turns Motion into Color

The canvas is divided into cells measuring 70 × 70 pixels.

Every frame, the simulation counts the particles in each cell and calculates their average RGB color:

\[
C_{\text{cell}} = \frac{1}{n}\sum_{i=1}^{n} C_i
\]

Here, \(n\) is the number of particles in the cell, and \(C_i\) is the color of an individual particle.

The cell is rendered using this average color, reduced to one-sixth of its brightness. Individual particles are drawn afterward, allowing them to remain visible against the shared color field.

This creates two simultaneous representations of the same system:

- The particles show fine-grained movement.
- The grid shows collective behavior.

A group of differently colored particles can create a new color simply by occupying the same cell. As they separate, that collective color changes again.

### 3. Color Has Memory

Without smoothing, every cell would change color abruptly whenever a particle crossed a boundary. Instead, each cell maintains its own persistent color.

On every frame, the current color moves 2% of the remaining distance toward the newly calculated average:

\[
C_{t+1} = C_t + 0.02(C_{\text{target}} - C_t)
\]

This is exponential smoothing.

It gives the grid a kind of visual inertia. The colors do not immediately follow every change in particle distribution; they lag behind slightly, leaving soft traces of the system's recent history.

At a constant frame rate of 60 frames per second, the color difference takes approximately 0.58 seconds to fall to 10% of its initial value, or about 1.15 seconds to fall to 1%.

Empty cells gradually return toward the dark background color instead of instantly disappearing.

The result is a field that remembers where activity has been without permanently preserving old patterns.

### 4. A Self-Organizing Force Field

Each particle receives an acceleration determined by the number of particles in its current cell.

Let \(n\) be the occupancy of the cell and \(i\) the particle's index in the array. The phase is calculated as:

\[
\theta_i = n + i \cdot b_i \cdot 10^{-8}
\]

where \(b_i\) is the particle's individual blinking parameter.

The acceleration is then:

\[
a_x = 0.07\sin(\theta_i)
\]

\[
a_y = 0.07\cos(\theta_i)
\]

The sine and cosine functions produce perpendicular components of a rotating direction. For a fixed phase, the acceleration vector has a magnitude of approximately 0.07.

The important detail is that the phase depends on local particle density. As particles enter or leave a cell, the force changes. That changes their trajectories, which changes the density again.

This creates a feedback loop:

1. Particles move through the grid.
2. Their distribution determines local density.
3. Density determines their acceleration.
4. Acceleration changes their movement.
5. Their new positions create a different density field.

No particle explicitly knows how to form a pattern. The patterns arise from the interaction between movement and local occupancy.

Because the density is an integer and the trigonometric functions are periodic, different cell populations can produce different directional tendencies. The force field is structured, but it is not a simple attraction toward a center or a straightforward diffusion process.

### 5. Momentum and Friction

The particles retain their velocities from frame to frame. Acceleration changes velocity, and velocity changes position.

The friction coefficient is `0.97`, but friction is applied in steps: once for each whole-number threshold exceeded by the particle's speed.

For speed \(v\), the number of applications is:

\[
k = \lfloor v \rfloor
\]

The velocity components are multiplied by \(0.97^k\).

This creates a nonlinear damping rule:

- Below a speed of 1, no friction is applied.
- Above 1, one application is applied.
- Above 2, two applications are applied.
- Above 3, three applications are applied.

This is not conventional continuous drag. It allows momentum to persist at lower speeds while applying stronger damping to faster particles.

Particles that cross a canvas boundary reappear on the opposite side. The field is effectively a continuous, wrapped space rather than a box with walls.

### 6. Independent Blinking

Every particle has a blinking parameter:

```javascript
blink: Math.floor(105 - Math.random() * 6)
```

This produces integer values from 100 through 104. Each particle also receives a random phase between zero and \(2\pi\).

Its blinking angular frequency is calculated as:

\[
\omega_i = 0.001 + (255-b_i)\cdot0.000025
\]

With the chosen parameter range, this gives angular frequencies from 0.004775 to 0.004900 radians per millisecond.

The brightness oscillates according to:

\[
B_i(t) = 0.5 + 0.5\sin(\omega_i t+\phi_i)
\]

The final opacity is:

\[
\alpha_i(t) = 0.05 + 0.95B_i(t)
\]

Consequently, each particle varies between 5% and 100% opacity. The random phase prevents all particles from reaching their brightest or dimmest points together.

The frequencies are close to one another, so the particles have similar blinking periods, but they do not share exactly the same timing. Their angular frequencies correspond to periods of approximately 21.4–21.9 minutes per complete cycle.

This means the particles do not actually blink rapidly. During a short viewing session, they appear almost steady, with only very gradual changes in brightness. Much of the apparent glitter comes from changing positions, colors, overlaps, and glow rather than rapid individual blinking.

The distinction matters: the current parameters create a slow shimmer rather than a fast flashing effect.

### 7. Layered Light and Glow

Each particle is drawn using several overlapping rectangles:

- A bright 3 × 3 pixel core.
- Two medium-sized shapes with 20% of the particle's current opacity.
- Two broad shapes with 5% of the particle's current opacity.

The broad shapes spread light over a larger area. The central core preserves a point of brightness, while the surrounding shapes blend nearby particles into soft patches of light.

The dark background provides contrast for the luminous regions.

Together, these layers create the impression of light bleeding through a granular surface. The particles are discrete in the simulation, but their overlapping glows make the visual result feel continuous.

## Why the Patterns Feel Rhythmic

The effect comes from the interaction of several timescales, not from a single repeating animation.

**Local density creates spatial rhythm.** Each grid cell produces a force based on its occupancy. Small changes in particle count can change the force direction, creating regions of correlated movement alongside areas that behave differently.

**Momentum creates temporal continuity.** Particles do not teleport toward a new target. Their velocities persist, allowing structures to travel, stretch, and dissolve over time.

**Color smoothing creates visual afterimages.** The grid retains some memory of its previous colors. The color field therefore evolves more slowly than the individual particle distribution.

**Independent phases prevent synchronized blinking.** Particles do not all brighten together. Their different phases and slightly different frequencies prevent a single global pulse from dominating the scene.

**Glow merges nearby events.** Overlapping light layers visually connect particles that would otherwise look like unrelated points.

Together, these processes create a visual polyrhythm: local changes, slower spatial reorganizations, and even slower color transitions coexist on the same canvas.

The eye perceives this as a kind of organic beat. It is not a fixed musical rhythm, but a continuous interplay between repetition and variation.

## Parameters

| Parameter | Current value | Role |
|---|---:|---|
| Particle count | 8,000 | Density and visual complexity |
| Grid size | 70 px | Scale of collective color patterns |
| Force | 0.07 | Strength of particle acceleration |
| Friction | 0.97 | Damping applied above speed thresholds |
| Index influence | 0.00000001 | Tiny particle-specific variation in force phase |
| Color mix | 0.02 | Speed of cell color transitions |
| Bright particles | Approximately 70% | Luminous texture |
| Dark particles | Approximately 30% | Contrast and visual depth |
| Canvas blur | None | Preserves the rendered shapes |

The simulation adapts the canvas to the browser window and wraps particles across the edges.

## The Underlying Idea

This experiment is built around a simple principle:

**Complexity does not have to be explicitly designed; it can emerge from the interaction of simple rules.**

The particles have no destinations. The cells have no predefined colors. The glow follows no predetermined path. Instead, local interactions continually change the state of the system, and the visual result is reconstructed from that state on every frame.

The field is never entirely stable, but it is not completely random either. Its structure is continually being created and destroyed.

That balance is what makes it interesting to watch: patterns form, hold together briefly, and dissolve into new patterns, like foam moving across the surface of water.

The intention is not to display information or tell a story. It is to create a visual environment that can be watched without needing to solve anything—a slow, luminous process in which the eye can wander, settle, and wander again.

---

*Particle Foam is an experimental generative-art sketch built with JavaScript and the HTML Canvas API. It uses no external libraries or assets.*
