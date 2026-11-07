# Build a Slingshot Launcher in Scratch

### (The Angry Birds "pull back and let go" mechanic)

By the end of this tutorial you'll have a dinosaur you can pull back with
your mouse, aim, and fling across the screen — and it'll make a rising
*stretching* whine as you pull back, plus a thump when it lands.

The cool part about how this is built: **you never press a key.** You just click,
drag, and release. And the moment the shot is over, it reloads itself and is
ready to go again, forever.

We'll build it in four pieces:

1. **The Setup** — two numbers that control how the game feels
2. **The Aimer** — a helper sprite that follows your mouse and measures your pull
3. **The Launcher** — the dinosaur, which reloads, waits, and flies
4. **The Sound** — a separate script that whines while you stretch

Read the whole thing once, then build it. Don't skip the "Why?" boxes — those are
the parts that make this click.

---

## Part 0: The big idea

Here's the trick that makes a slingshot work. It's not complicated, it just has a
fancy name: **a vector**.

When you pull a real slingshot back, two things matter:

- **Which direction** you pulled
- **How far** you pulled

In Scratch, you can describe both of those with just two numbers: how far apart
the dinosaur and your mouse are, **sideways** and **up-and-down**. We'll call
them `difx` and `dify` (short for "difference in x" and "difference in y").

```
    DINOSAUR *
             |  \
     dify -> |    \   <- this whole diagonal is your "pull"
             |      \
             +--------* MOUSE
                difx
```

If you pull the mouse far to the right and a little down, then `difx` is a big
number and `dify` is a small one — so the dinosaur should fly fast to the left
and drift slightly up. **That's the entire mechanic.** Everything else below is
just plumbing.

> **Why two numbers instead of an angle and a distance?**
> Because "add difx to my x position" is one easy block. Angles would mean sine
> and cosine, and we don't need them. Splitting motion into *sideways* and
> *up-down* is how real game engines do it too.

---

## Part 1: Make your variables

Click the orange **Variables** category, then **Make a Variable** for each one
below. Every single one should be **"For all sprites"** — that matters, because
two different sprites need to read and write them.

| Variable | What it holds |
|---|---|
| `difx` | How far the mouse is from the dinosaur, sideways |
| `dify` | How far the mouse is from the dinosaur, up and down |
| `dist` | The straight-line distance from dinosaur to mouse (how hard you pulled) |
| `speedX` | How many steps the dinosaur moves sideways each frame |
| `speedY` | How many steps the dinosaur moves up/down each frame |
| `gravity` | How much the dinosaur's upward speed shrinks each frame |
| `launchPower` | A dial for how strong the slingshot is |

When you're done, **uncheck all the little boxes** next to them in the palette so
they don't clutter up your stage. (Checking one back on is a great way to debug
later — you can literally watch the number change while you drag the mouse.)

---

## Part 2: The Setup script

Put this on the **dinosaur**, all by itself:

```
when green flag clicked
    set [gravity     v] to (-1)
    set [launchPower v] to (0.1)
```

That's it. Two blocks.

> **Why is this its own separate script?**
> You could jam these into the top of the big launch script, but keeping them
> apart is a nice habit. These are the two **settings** for your whole game, and
> now they live in one obvious place. When you want a floatier shot, you know
> exactly where to go. Programmers call this "configuration," and separating it
> from the action is a real design choice — not just tidiness.
>
> It also means they get set **once** at the start, instead of being re-set over
> and over on every single shot. Same result here, but less wasted work.

---

## Part 3: The Aimer sprite

The **Basketball** is your aiming handle. Its whole job is:

> "Sit where the mouse is, and constantly report how far I am from the dinosaur."

Build this on the **Basketball**:

```
when green flag clicked
forever
    go to x: (mouse x) y: (mouse y)

    set [dist  v] to (distance to [AngryBird v])
    set [difx  v] to ((x position of [AngryBird v]) - (x position))
    set [dify  v] to ((y position of [AngryBird v]) - (y position))

    if <(144) < (distance to [AngryBird v])> then
        set [difx v] to ((difx) * ((144) / (distance to [AngryBird v])))
        set [dify v] to ((dify) * ((144) / (distance to [AngryBird v])))
```

Let's walk through it.

**`go to x: (mouse x) y: (mouse y)`** — the sprite becomes your cursor. Now
"where the mouse is" and "where this sprite is" are the same thing, which lets us
use Scratch's handy `distance to` block.

**The two `set` blocks** — subtract to get the gap. Notice the order: it's
`dinosaur minus mouse`, not `mouse minus dinosaur`. That's on purpose! It means
that when you pull the mouse to the **left** of the dinosaur, `difx` comes out
**positive**, so the dinosaur flies **right** — away from your pull, exactly like
a real slingshot. Flip the subtraction and it'll fly *toward* your mouse. (Try
it. Being able to explain a bug is better than never having one.)

**The `if` block** — this is the **maximum stretch**. A real slingshot can only
pull back so far. Without this, you could drag your mouse to the far corner of
the stage and launch at a ridiculous speed.

> **How does that squishing math work?**
> Say the limit is 144 but you pulled 288 — twice too far. Then `144 / 288` is
> `0.5`, and multiplying both `difx` and `dify` by `0.5` cuts the pull in half.
> It's now exactly 144 long, but pointing the *same direction*. That's the
> important part: scaling both numbers by the same amount changes the **length**
> without changing the **aim**.
>
> `144` is just a number *you* picked. Bigger = longer slingshot = harder shots.
> If you change it, change it in **all three places** or the math goes crooked.

---

## Part 4: The Launcher (the dinosaur)

Here's the main event. The plan in plain English:

1. Snap back to the slingshot
2. Wait for the player to press the mouse button *(they're grabbing the band)*
3. Tell the sound script to start whining
4. Wait for them to let go
5. Turn the pull into speed, and fly
6. Thump — **then go back to step 1, forever**

```
when green flag clicked
forever
    go to x: (-99) y: (-59)

    wait until <mouse down?>
    broadcast [launchSound v]
    wait until <not <mouse down?>>

    set [speedX v] to ((launchPower) * (difx))
    set [speedY v] to ((launchPower) * (dify))

    repeat (77)
        set x to ((x position) + (speedX))
        set y to ((y position) + (speedY))
        change [speedY v] by (gravity)

    start sound [Chomp v]
```

### The flight

```
    set [speedX v] to ((launchPower) * (difx))
    set [speedY v] to ((launchPower) * (dify))
```

This is the handoff: the pull you measured becomes actual speed. `launchPower` of
`0.1` means a full 144-step pull becomes a speed of about 14 steps per frame.

Then the loop is the physics, and there are only three lines of it:

- **`set x to (x position + speedX)`** — drift sideways. `speedX` never changes
  during flight, so horizontal speed stays constant. (Real life has air
  resistance. We're ignoring it, and honestly it feels better this way.)
- **`set y to (y position + speedY)`** — drift vertically.
- **`change speedY by gravity`** — **this one line is gravity.** Since `gravity`
  is `-1`, `speedY` shrinks by 1 every single frame.
---

## Part 5: The stretch sound

This is its own little script on the dinosaur

```
when I receive [launchSound v]
    set [pitch v] effect to (0)

    repeat until <not <mouse down?>>
        start sound [Chomp v]
        change [pitch v] effect by (10)

    set [pitch v] effect to (0)
```

### ⚠️ The sticky-pitch trap

See those two `set [pitch v] effect to (0)` blocks? **You need both**, and this
is the single most common way this script goes wrong.

The pitch effect is **sticky**. It isn't a one-time thing — once you crank it up,
it *stays* cranked on that sprite until you explicitly clear it. So if you only
have `change [pitch v] effect by (10)` and never reset:

- Shot 1: whine climbs to +300. Nice!
- The landing thump plays... at +300. Squeaky.
- Shot 2: the whine *starts* at +300 and climbs to +600.
- Shot 10: everything sounds like a dying mosquito, forever.

The fix is the two resets:

- **`set [pitch v] effect to (0)` at the top** — every stretch starts from the
  same low pitch, so every shot sounds the same.
- **`set [pitch v] effect to (0)` at the bottom** — when you release, the pitch
  drops back to normal, so the landing thump sounds like a thump instead of a
  squeak.

> If your project is already getting squeakier every shot, this is why. Drop in
> those two blocks and it's fixed.

