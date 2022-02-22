<!-- Inspiration for README's: https://github.com/matiassingers/awesome-readme  -->

# Generator

## Contents

1. [Introduction](#introduction)
1. [Magic Square](#magic-square)
1. [Desired goal times](#desired-goal-times)

## Introduction

The generator represents a 5x5 bingo board as an array of 25 squares, and aims to populate the positions 0 to 24 of the
array with a goal. The first 5 elements of the array correspond to the squares of row 1, the next 5 to the squares of
row 2, et cetera.

## Magic Square

The first step to generating a card is calculating a 5x5 [magic square](https://en.wikipedia.org/wiki/Magic_square). In
a magic square, the sum of the numbers in each row, column and diagonal is the same. For bingo cards, the number **1**
to **25** are used to fill each position of the square, all only appearing once. The sum of each possible row, column or
diagonal is equal to **65**.

The code to generate a magic square can be found in `magicSquare.ts`. Based on the seed, the
function `magicSquareNumber()` calculates what number should go in a given position of the square. It gets called
by `generateMagicSquare()`, which loops over positions 1 to 24 to calculate each magic square number.

### Example

With seed `112233`, the following magic square numbers get generated:

```ts
[21, 15, 19, 3, 7, 4, 8, 22, 11, 20, 12, 16, 5, 9, 23, 10, 24, 13, 17, 1, 18, 2, 6, 25, 14]
```

Formatted as an actual square:

|   21   |   15   |   19   |   3    |   7    |
|:------:|:------:|:------:|:------:|:------:|
| **4**  | **8**  | **22** | **11** | **20** |
| **12** | **16** | **5**  | **9**  | **23** |
| **10** | **24** | **13** | **17** | **1**  |
| **18** | **2**  | **6**  | **25** | **14** |

## Desired goal times

### Difficulty

The term **'difficulty''** is being used in a slightly confusing way in bingo lingo. Counterintuitively, the difficulty
of a goal is related to its length, and has nothing to do with how hard or easy the goal is to complete. The latter is
of a goal is related to its length, and has nothing to do with how hard or easy the goal is to complete. The latter is
being taken into account by the timings sheet, and is called the '**skill**' of a goal.

The number on each tile of the generated magic square represents the difficulty of that tile. That means that the tile
with a 1 should get the shortest goal, and the tile with 25 should get the longest. Because of the properties of the
magic square, this approach results in rows that are all of similar length.

Note that from now on, the term 'square' will indicate one tile of the board.

### Time per difficulty

Now, to know what goals can go in what squares, we need to convert the difficulty of each square to an actual time in
minutes. That is done by multiplying with the `timePerDifficulty` constant. All constants that the generator uses are
defined in `definitions.ts`. There are profiles with different constants for each type of bingo card.
In `types/profiles.ts` you can find a short explanation on each parameter.

### Desired time

Each difficulty gets multiplied by the timePerDifficulty to get the **desired time** of the square. The desired time is
the ideal amount of time a goal on that square should take to complete. The generator is allowed to deviate from it a
little; the `offset` constants define how much. More on that later.

The function `mapDifficultyToSquare()` maps a difficulty to a `Square` object, which contains the
calculated `desiredTime`, and the unchanged `difficulty` for reference.

### Example

For normal bingo cards, the `timePerDifficulty` is equal to **0.75** (at the time of writing this). Continuing the
earlier example, the first square with number **21** gets a **desired time** of  **15.75** (`21 * 0.75`), i.e. 15m45s.
The second square gets a desired time of **11.25** (`15 * 0.75`). Doing this for all squares results in the following
**desired times** (in minutes):

|  15.75   |  11.25  |  14.25   |   2.25    |   5.25    |
|:--------:|:-------:|:--------:|:---------:|:---------:|
|  **3**   |  **6**  | **16.5** | **8.25**  |  **15**   |
|  **9**   | **12**  | **3.75** | **6.75**  | **17.25** |
| **7.5**  | **18**  | **9.75** | **12.75** | **0.75**  |
| **13.5** | **1.5** | **4.5**  | **18.75** | **10.5**  |

## Population order

It's almost time to start picking goals. But first, the generator needs to know *in which order* each square has to be
populated with a goal. This happens in the `generatePopulationOrder()` function, which returns a list of indices. The
order is as follows:

1. The 3 squares with the **highest difficulty** *always* get populated first (difficulty **25**, then **24**, then **
   23**)
2. Then the **center** square of the board
3. Then the squares on the **diagonals** (tlbr/bltr), in random order
4. Then the all the other squares, in random order

Note that for steps 2-4, it can occur that some of the squares were already picked in step 1, then they get ignored in
these steps.

The reason that the squares with the highest difficulty get picked first, is that they will have the longest desired
times, meaning they will be populated with the longest goals. But there are significantly less long goals to pick from.
If you pick them late, there might not be any options left that do not conflict with already populated squares.

For similar reasons, the center square and diagonals get populated before the rest of the board. These squares appear in
the most rows, so goals on those squares deal with more restrictions caused by goals in the rows they're in. Generation
is more likely to be successful if you pick these first.

### Example

On our example board, the **population order** would be:

```ts
[23, 16, 14, 12, 8, 24, 20, 18, 6, 4, 0, 10, 1, 19, 22, 5, 21, 17, 7, 15, 3, 2, 11, 13, 9]
```

Index **23** belongs to the square with difficulty **25** (row5/col4), index **16** belongs to the square with
difficulty **24** (row4/col2) and index **14** belongs to the square with difficulty **23** (row3/col5).

The next index is **12**, which is the center square (happens to have difficulty 5). Then the diagonal square indices
follow in random order (**8**, **24**, **20**, **18**, **6**, **4**, **0**). Note that index **24** is also on a
diagonal, but it already appeared earlier in the list because its square has difficulty **24**.

The next indices belong to all the other squares, in random order.

## Picking goals