# Build a Slingshot Launcher in Scratch

### (The Angry Birds "pull back and let go" mechanic)

Hi! By the end of this tutorial you'll have a dinosaur you can pull back with
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

### Why `forever` is the clever part

This is the best idea in the whole project, so let's slow down on it.

The obvious way to build a launcher is `when [a] key pressed`. But that has an
annoying problem — and it's a genuine Scratch quirk worth knowing:

> **`when [key] pressed` will NOT restart a script that's already running.** It
> just ignores you. So if the dinosaur is still mid-flight, or still sitting
> there waiting for your click, pressing the key again does absolutely nothing.
> The launcher feels broken and you can't figure out why.

The `forever` loop sidesteps that entirely. Look at the shape of it: the **last**
thing in the loop is the landing thump, and then it wraps around to `go to x:-99
y:-59` — the reload. The launcher **puts itself back together after every shot**.

So there's no key to press, no script to restart, and no way to get stuck. Click,
drag, release, repeat. The player can fire as fast as they can click.

This is a pattern you'll use constantly: **a `forever` loop with `wait until`
blocks inside it** is how you build something that handles one event after
another, in order, over and over. The `wait until` blocks are the pause points
where the loop sits politely and does nothing until the player acts.

### `broadcast` — running two things at once

**`broadcast [launchSound v]`** is a message. It shouts "launchSound!" to the
whole project, and any script with a matching `when I receive` hat wakes up and
starts running.

The crucial detail: the plain `broadcast` block **does not wait**. It fires the
message and immediately moves to the next block. So the instant you press the
mouse:

- the **launch script** carries on to `wait until <not <mouse down?>>`
- the **sound script** starts whining

...and those two run **side by side, at the same time.**

> **Careful:** there's also a `broadcast [ ] and wait` block. If you use that one
> by mistake, your launch script will freeze until the sound script completely
> finishes — and since the sound script only finishes when you *release* the
> mouse, you'd get a weird stutter. Use plain `broadcast` here.
>
> This is called **concurrency** — two things happening at once. It's a big deal
> in real programming, and Scratch lets you do it by just... making two scripts.

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
  is `-1`, `speedY` shrinks by 1 every single frame. Watch what that does:

| Frame | `speedY` | What you see |
|---|---|---|
| 1 | 10 | rising fast |
| 5 | 6 | still rising, slower |
| 10 | 1 | almost stopped — top of the arc |
| 11 | 0 | the peak |
| 15 | -4 | falling |
| 20 | -9 | falling fast |

You never told it to make an arc. You just said "lose a little upward speed every
frame," and the arc *appears*. That's the whole idea behind game physics, and
you've now written it.

**Why `repeat (77)`?** That's your **flight time**. The loop runs about 30 times
a second, so 77 frames is roughly two and a half seconds of flight, and then the
shot is over no matter what. It's a nice round-ish number that someone picked
because it felt right — which is a completely legitimate way to tune a game.

Make it `repeat (200)` and shots hang around much longer. Make it `repeat (20)`
and the dinosaur barely gets going before it resets.

> **A tidier option:** try swapping `repeat (77)` for
> `repeat until <touching [edge v]?>`. Then the flight ends when the dinosaur
> actually hits something instead of when a timer runs out. Both are fine — the
> counter is simpler, the `touching` version is smarter.

**`start sound [Chomp v]`** at the very end is the landing thump, played once the
flight finishes. Then the loop wraps around and reloads.

---

## Part 5: The stretch sound

This is its own little script on the dinosaur. It's only three blocks, and it
does something genuinely cool:

```
when I receive [launchSound v]
    set [pitch v] effect to (0)

    repeat until <not <mouse down?>>
        start sound [Chomp v]
        change [pitch v] effect by (10)

    set [pitch v] effect to (0)
```

**`repeat until <not <mouse down?>>`** — "keep doing this the whole time they're
holding the button down." Read it out loud: *repeat until NOT mouse down*. It's a
double negative, which is awkward, but it's how Scratch says "while."

**`start sound [Chomp v]`** fires a short sound every frame — about 30 times a
second. Normally that would be a horrible mess, and here's the trick that turns
it into something good:

**`change [pitch v] effect by (10)`** nudges the pitch up a notch every single
frame. So each copy of the sound is slightly higher than the one before it, and
they pile up into one continuous **rising whine**. The longer you hold, the
higher and tighter it gets — exactly like a rubber band stretching.

That's the whole effect: a boring sound, fired rapidly, with the pitch climbing.
No special audio tricks. You can hear *tension* in it.

> **Try this:** change the `10` to `3` for a slow, heavy, creaky stretch. Change
> it to `40` for a cartoonish *zip*. This one number completely changes the
> personality of your slingshot.

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

---

## Part 6: Try it, then break it

Click the green flag. Press and hold the mouse button, drag away from the
dinosaur, and let go. Then do it again — notice you never had to reset anything.

Now go mess with the dials. This is the actual point of the exercise:

| Change this | To this | And see |
|---|---|---|
| `launchPower` | `0.3` | A cannon |
| `launchPower` | `0.02` | A very sad dinosaur |
| `gravity` | `-0.3` | Moon gravity, long floaty arcs |
| `gravity` | `-3` | A heavy rock |
| `gravity` | `0` | No gravity — it flies forever in a straight line |
| `gravity` | `1` | Upside-down gravity. It flies *up* and never comes back |
| `repeat (77)` | `repeat (200)` | Much longer flights |
| the `144` limit | `250` | A giant slingshot (change all three!) |
| `change pitch by (10)` | `(3)` | A slow, heavy creak |
| `change pitch by (10)` | `(40)` | A cartoon *zip* |

Make a prediction *before* you click. Being wrong is where the learning is.

---

## Troubleshooting

**It flies toward my mouse instead of away.**
Your subtraction is backwards. It needs to be
`(x position of AngryBird) - (x position)`, with the dinosaur first.

**Everything sounds squeaky, and it gets worse every shot.**
You're missing the `set [pitch v] effect to (0)` blocks. See the sticky-pitch
trap in Part 5.

**The stretch sound keeps whining after I let go.**
Check that `change [pitch v] effect by (10)` is **inside** the `repeat until`
loop and that the loop's condition is `<not <mouse down?>>`. If the condition is
just `<mouse down?>`, the loop exits instantly and then never stops.

**It launches the moment I click, with no stretch.**
Your `wait until <not <mouse down?>>` is missing, or it's above the broadcast
instead of below it. Order matters: wait for the press, broadcast, *then* wait
for the release.

**It only fires once, then freezes.**
Something in your `forever` loop never finishes. The usual culprit is a
`wait until` that can never come true, or `broadcast and wait` where you wanted
plain `broadcast`.

**It gets stuck on the edge of the screen.**
That's normal — Scratch won't let sprites leave the stage, so it pins there while
`repeat (77)` finishes counting. Wait a moment and it reloads on its own. The
`repeat until <touching [edge v]?>` swap from Part 4 fixes it properly.

**`difx` and `dify` stay at 0.**
Your Basketball script isn't running (click the green flag), or you made the
variables "For this sprite only" instead of "For all sprites." If it's the second
one, delete them and remake them — the dinosaur can't see the Basketball's
private variables.

**There are random loose blocks floating in my code area.**
Totally fine, and you probably have a few: a lone `direction` block, a
`think (direction of Wall)`, a `(backdrop # of Stage)`. Those are leftovers from
poking at things to see what they report — a completely normal way to explore.
They're not attached to any hat block, so they never run. Right-click → Delete
when you want to clean up. (Clicking one is a handy trick, by the way: click any
reporter block and Scratch shows you its value in a little bubble.)

---

## What's next: the Wall

There's a **Wall** sprite in the project that doesn't do anything yet. That's the
natural next step — the dinosaur flies, but nothing happens when it *arrives*.

The block you'll want is `<touching [Wall v]?>`, and the place to put it is
inside the flight loop. Something shaped like:

```
    repeat until <<touching [Wall v]?> or <touching [edge v]?>>
        set x to ((x position) + (speedX))
        ...
```

...and then do something fun when the loop ends. That's a whole project of its
own, though. Get the launching feeling right first.

---

## What you learned

Quietly, in about 25 blocks, you did all of this:

- Split motion into **x and y components** — the foundation of all 2D game movement
- **Clamped** a value to a maximum while preserving its direction
- Made **gravity** out of one `change by` block
- Built a **self-resetting loop** (`forever` + `wait until`) that handles events
  in order and never gets stuck
- Used a **broadcast** to run two scripts **at the same time**
- Separated your **settings** from your **action code**
- Mapped a game value onto **sound pitch** for feedback the player *feels*

That last one is real game design. The pitch rising as you pull isn't decoration
— it tells the player how hard they're about to shoot without a single number on
screen. Good games do this constantly, and most people never notice. Now you
will.
