---
title: "Level format"
categories: analysis
---

The Amiga version of _Populous_ features 495 (actually 1,001) procedurally
generated levels accessible by entering a level code. Each level has various
attributes, such as terrain type, enemy rating, and starting population.

A full list of level codes appears on a separate page,
[List of Populous levels](../data/level-list.html).

### Level code algorithm

Each level code is divided into a start, middle, and end segment. There are 32
possible values for each:

| #  | Start| Mid | End  |
|----|------|-----|------|
| 00 | RING | OUT | HILL |
| 01 | VERY | QAZ | TORY |
| 02 | KILL | ING | HOLE |
| 03 | SHAD | OGO | PERT |
| 04 | HURT | QUE | MAR  |
| 05 | WEAV | LOP | CON  |
| 06 | MIN  | SOD | LOW  |
| 07 | EOA  | HIP | DOR  |
| 08 | COR  | KOP | LIN  |
| 09 | JOS  | WIL | ING  |
| 10 | ALP  | IKE | HAM  |
| 11 | HAM  | DIE | OLD  |
| 12 | BUR  | IN  | PIL  |
| 13 | BIN  | AS  | BAR  |
| 14 | TIM  | MP  | MET  |
| 15 | BAD  | DI  | END  |
| 16 | FUT  | OZ  | LAS  |
| 17 | MOR  | EA  | OUT  |
| 18 | SAD  | US  | LUG  |
| 19 | CAL  | GB  | ILL  |
| 20 | IMM  | CE  | ICK  |
| 21 | SUZ  | ME  | PAL  |
| 22 | NIM  | DE  | DON  |
| 23 | LOW  | PE  | ORD  |
| 24 | SCO  | OX  | OND  |
| 25 | HOB  | A   | BOY  |
| 26 | DOU  | E   | JOB  |
| 27 | BIL  | I   | ER   |
| 28 | QAZ  | O   | ED   |
| 29 | SWA  | U   | ME   |
| 30 | BUG  | T   | AL   |
| 31 | SHI  | Y   | T    |

Although this technically allows 32 x 32 x 32 = 32,768 possible level codes,
only 1,001 codes are accepted. To see how this works, we must begin with the
game's random number generator.

```
MOVE.W  _seed,D0   ; 16-bit seed value
MULU    #$24a1,D0  ; x 9,377
ADDI.W  #$24df,D0  ; Add 9,439 (only use last 16 bits)
BCLR    #$F,D0     ; Clear top bit so it can't be negative
MOVE.W  D0,_seed   ; Save the new value to give a different result next time
RTS
```

This is a 
[linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator),
an efficient pseudorandom number generator. It produces numbers which are
deterministic, but unlikely to form any pattern that a player could predict, so
it seems random enough for a video game. It takes in a 16-bit seed value and
outputs a number between 0 and 32,767.

Each level has a "battle number" (i.e. the level number shown to the player) and
an internal world code. The code is a 16-bit number. The top bit is ignored, and
the remaining 15 bits are divided into three sets of 5 bits. Each 5-bit number
is read as a number between 0 and 31, and represents one of the code segments in
the table above. The lowest five bits represent the first segment, and so on.

Effectively, the level codes are calculated by multiplying the battle number by
5 and feeding the result into the random number generator once. Level 0 is
therefore calculated as 0 x 5 x $24a1 + $24df = $24df, divided into segments
(31, 6, 9), or SHISODING. A special case also allows world 0 to be accessed
using the code GENESIS, which is its default name.

_Populous_ is coded to cap out at level 1,000, EOADIEPERT. Codes above this will
not be recognized. In normal play, the game actually caps out at level 494,
WEAVUSPERT. Completing WEAVUSPERT will give this unique message:

```
WELL DONE YOU HAVE CONQUERED EVIL
THE BATTLE IS OVER BUT TRY SHISODING
```

SHISODING is, as mentioned, is an alias for level 0, GENESIS. Levels 495 to
1,000 will generate oddly: their traits are based on the last level code you
entered, and their layout may be based on another map.

### Level progression

When you complete a level in Conquest mode, you may progress multiple levels
depending on your score. At minimum, you progress to the next numbered level.
The maximum level is 494, WEAVUSPERT.

(see scoring, TBA).

### Level traits

In addition to map layout, each Conquest map has certain traits: terrain type,
enemy reaction speed, renemy rating, land building rules, swamp rules, water
rules, starting population for each player, and which of the powers you and the
opponent have access to.

Level traits are stored in the file `level.dat`, a 990 byte file which stores 99
ten-byte entries. Each entry covers five consecutive levels, meaning that level
traits only change every five numbered levels. For example, levels 0 to 4 all
have the same traits (Grass Planes, Build on People, Swamps are Bottomless,
etc), levels 5 to 9 all share traits (Desert, Swamps Shallow, etc) and so on.

The significance of each byte is as follows:

 Byte  | Use
-------|----------
 00    | Enemy rating
 01    | Enemy reaction speed
 02    | Powers available to enemy
 03    | Powers available to you
 04    | Game mode (build rules, swamps, water)
 05    | Terrain type
 06    | Your starting population
 07    | Enemy starting population
 08-09 | Random two-byte seed offset

#### Byte 00: Enemy rating

A number from 1 to 10 ranking the enemy's rating, i.e. difficulty. 1 is the
strongest, and 10 is the weakest. The player is never shown the actual number,
only one of five categories. However, you can see it by putting the game in
Custom mode, switching to Evil and clicking the weighing scales icon, which will
show you rating and speed as sliders.

- 1-2 Very Good
- 3-4 Good
- 5-6 Average
- 7-8 Poor
- 9-10 Very Poor

#### Byte 01: Enemy reaction speed

A number from 1 to 10 ranking the enemy's reaction speed, also referred to as
Aggression.

- 1-2 Very Fast
- 3-4 Fast
- 5-6 Medium
- 7-8 Slow
- 9-10 Very Slow

#### Byte 02-03: Powers available

Byte 2 is a bitfield selecting which powers are available to the enemy, and byte
3 is the equivalent for the player.

- Bit 0: Earthquake
- Bit 1: Swamp
- Bit 2: Knight
- Bit 3: Volcano
- Bit 4: Flood
- Bit 5: Armageddon

Armageddon isn't listed on the new game screen due to lack of space, but it's
still available.

#### Byte 04: Game mode

This is a bitfield detailing various game options.

- Bit 0: Water
  - 0: Water is harmful
  - 1: Water is fatal
- Bit 1: Swamp
  - 0: Swamps shallow
  - 1: Swamps bottomless
- Bit 2: Build
  - 0: Can build
  - 1: Cannot build
- Bit 3: Build
  - 0: Build up and down
  - 1: Only build up
- Bit 4: Build
  - 0: Build near people
  - 1: Build near towns

In practice, bits 2, 3, and 4 are mutually exclusive in the Conquest data set,
and only one of the three will be set, or none at all. However, they can be set
independently in custom mode.

#### Byte 5: Terrain type

- 00: Grass Plane
- 01: Desert
- 02: Snow and Ice
- 03: Rocky

Five additional terrain types are available with the expansion disk _The
Promised Lands_ and one on _The Final Frontier_.

#### Byte 6-7: Starting population

Byte 6 holds your starting population of walkers. Byte 7 holds the enemy's
starting number of walkers. The maximum starting value is 30 for both players.

In certain levels where the starting population is high, a bug occurs where a
large number of enemy walkers disappear and are quickly supplanted by the
player's walkers. See bugs (TBA).

#### Byte 8-9: Seed

A random 16-bit number. It is added to (battle number mod 8) to set the random
seed at the start of the round. The main effect is presumably to select unique
and interesting map layouts.

### List of levels

A full list of all 1,001 level codes appears on a separate page,
[List of Populous levels](../data/level-list.html).

### Notes

Additional thanks goes to Kenneth Fuchs for
[PopulousKeys](https://aminet.net/package/game/hint/PopulousKeys),
a list of all 495 main Populous codes from 1991, which previously appeared on
Fish Disk 531. Although I calculated my list independently, the PopulousKeys
data set was helpful in decoding the format of `level.dat` and double-checking
my results.
