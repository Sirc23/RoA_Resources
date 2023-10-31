# Thunderline Hitboxes
## Overview
Have you ever designed an attack where the hitbox needs to connect between two points in space? Maybe it's a beam attack that homes in on an opponent, or maybe it's a thin, long, melee attack that can be angled in strange increments? And then you go to implement it and realize there isn't an easy way to do that without creating a _bunch_ of small hitboxes. Next thing you know you end up with something like this:

![](https://i.imgur.com/sHZqWe0.png "na_pic")

Thankfully those days are now over! Thanks to __THUNDERLINE HITBOXES__, you can create a rectangular hitbox of not only any length or width, but also ___rotation___!

(insert gif of Sandbert creating a basic rotated hitbox)

These hitboxes are very flexible, and function very similarly to normal hitboxes with a couple key differences. Read on to learn about how they work as well as their capabilities.

## TL;DR - I want to make my own thunderline hitboxes!

Download this .zip file and merge the contents of each file into your own character's scripts. If you're not sure where they go, putting them at the top of the file is fine for most situations.

Once that's there, pick your hitbox, open up its definition file (`scripts/attacks/`), and modify / add the following attributes:

1. Set `HG_HITBOX_TYPE` to `2` (it _must_ be a projectile hitbox, this is a Rivals limitation)
2. Set `HG_SHAPE` to `3` (this is custom for thunderline hitboxes and is technically optional if you know what to change)
3. Set `HG_SIRC_INITIAL_ROTATION_ANGLE` to whatever angle you want the hitbox to be (can be any integer or decimal)

And there you go! You've just transformed an ordinary projectile into a thunderline hitbox!

## Key points for tweaking the code
If you want to experiment with these new hitboxes, here's a list of what I've discovered so far. If you learn something interesting, DM @sirc on Discord and I will be sure to update this list (with credit).

* Hitboxes must be projectiles. Melee hitboxes don't work. Thankfully, you can make a projectile act just like a melee hitbox  with enough effort.
* Unfortunately it seems like `image_yscale` (or `HG_HEIGHT`) don't work. The height seems hard-coded to 32px. Trying to find a solution that isn't "stack more projectiles" - that's what this is trying to avoid!
* The `attack` variable must be set to `AT_NSPECIAL_2`. If you have existing code that checks for hitboxes with this value, you will need to make some changes. I have added a variable called `orig_attack` that stores its original value.
* `hitbox_timer` must always be between values `12` and `20` for the hitbox to be active. Absa's thunderline is hard-coded by the engine to be active during these values. This also means that these hitboxes have the unique property of being able to exist in the world without damaging players.
* `length` (the hitbox's total lifetime) must be greater than `hitbox_timer` at all times, or else the hitbox despawns. That's just how Rivals works, but you don't really think about it until you set `HG_LIFETIME` to a number under `12` and wonder why your hitbox stops spawning!
* The angle of the hitbox is determined by the `shock_angle` variable. This value can be changed in real-time! ___Yes, spinning hitboxes!!___
* The width of the hitbox is determined by the `hbox_width` variable, NOT `image_xscale`.
* Hitboxes are hard-coded to be rectangular. Other `HG_SHAPE`s don't work, I've tried.
* The best way to uniquely identify a thunderline hitbox is the following:

        with(pHitBox)
        {
            if(variable_instance_exists(self, "is_thunderline")) {/*Do stuff*/}
        }
    
    The `is_thunderline` variable was custom-added just for this purpose. Alternatively, if you want to filter for any thunderline hitbox (including Absa's actual thunderline), you can test it this way:
    ```
    with(pHitBox)
    {
        if(select == 8 && type == 2 && attack == AT_NSPECIAL_2) {/*Do stuff*/}
    }
    ```

* The `fx_created` variable can be used to spawn a visual effect, but I have not experimented with how it works.
* `HG_SIRC_INITIAL_ROTATION_ANGLE` is obviously custom. By default it is set to `94` as an attempt to avoid conflict with other frameworks such as Munophone. If this causes strange interactions with certain characters, please DM me on Discord (@sirc) and I will take a look.

## How does it work?

You can see most of the magic happens in `hitbox_init.gml`. Basically, we need to trick the game into thinking...
* ...that the hitbox belongs to Absa
* ...that the hitbox is Absa's thunderline attack (Hold NSpecial while cloud is out)
* ...the attack is active for the amount of time we want it to be
 
That first one is easy to do, but was _not_ easy to figure out. Hitboxes have a variable called `select`, which is set to a unique index based on the character you've picked. I didn't spend the time to figure out all of its potential values, but al of Absa's hitboxes have this value set to `8`. Clairen's hitboxes, for example, use `12`.

The second one is easy too - just set the `attack` value to `AT_NSPECIAL_2`. Turns out the hitbox number doesn't seem to matter for our purposes (it does not check for it, but if you want to know, the thunderline itself is hitbox number `1`).

The third one is slightly trickier, but has a conventional solution. It turns out that the `HG_LIFETIME` property of thunderline is actually `24`, but the hitbox is only active for 8 frames. That means that there's a period of time before and after the active frames where a thunderline hitbox exists, but does not damage opponents. This is _not_ standard for hitboxes. In fact, thunderline is hard-coded to be active on frames 12-20 of the hitbox's lifetime. With this in mind, we can modify the `hitbox_timer` variable of our hitbox to always equal `12`, then manually keep track of when it should be active using another variable.

## I want a template character!
I'm working on a template character for this that will be published to the Workshop. It will be linked below once I have something ready. I'll try my best to include several different creative use cases for this tech. If you'd like to contribute, just shoot me a DM on Discord (@sirc) and I'll happily take a look.
