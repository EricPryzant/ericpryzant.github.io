---
layout: post
title: "Riddler Jenga from Feb. 19, 2021"
date: 2021-02-19
---
# Riddler Classic

https://fivethirtyeight.com/features/can-you-win-riddler-jenga/

In the game of Jenga, you build a tower and then remove its blocks, one at a time, until the tower collapses. But in Riddler Jenga, you start with one block and then place more blocks on top of it, one at a time.

All the blocks have the same alignment (e.g., east-west). Importantly, whenever you place a block, its center is picked randomly along the block directly beneath it. For example, the following animation shows Riddler Jenga towers that were randomly constructed before ultimately collapsing when the fifth, 10th and 15th blocks were placed. The block highlighted in red is the one above which the blocks were no longer balanced.

**On average, how many blocks must you place so that your tower collapses — that is, until at least one block falls off?**

(Note: This problem is not asking for the average height of the tower after any unbalanced blocks have fallen off. It is asking for the average number of blocks added in order to make the tower collapse in the first place.)

## Solution

We're going to attack this problem empirically to derive a solution through many many many simulations. We could absolutely derive an answer from theory by applying basic statistics but it's more fun and good practice to solve this through code.
```
## Initialize all necessary variables
import random
from tqdm import tqdm

length = 10
iterations = 1000000
output_msg = "\nSimulation results:\tAverage={avg}\tHighest={h}\tLowest={l}\tCumulative Total={ct}\tIterations={i}"
cumulative_height, avg, h, l = 0, 0, 0, 0
debug = False
```

The simulation is going to be split up into two main actions. The first function is used to create a new block to be added to the Jenga tower at a random new location. It has to be placed somewhere on top of the last block and should be placed randomly using a uniform distribution.

```
## Add a new block to the tower
def generate_new_block(last_block: float, length: int = 1) -> float:
    rand_start = last_block - length/2
    rand_end = last_block + length/2
    new_block = random.uniform(rand_start, rand_end)
    return new_block
```

The next function used in this simulation is to check whether or not the tower is stable. An unstable tower will, of course, fall down but this only happens when the weight of the blocks is centered outside of the platform that it rests upon. This means that when we place more than one block on the tower, everything above a given block must have it's center of mass between the left and right sides of the block. We can calculate the center of mass for a group of blocks by treating it as a single object and assuming that each block weighs the same. This allows us to simplify our calculations as we take the average location of the blocks in the group and use that as the center of mass. 

The code below will take a given tower of blocks along with the location of the new block then simulate the center of mass for each segment of the tower, starting from the bottom and working up. If the tower is balanced then the simulation can carry on. If not, then the simulated tower comes crashing down.
```
## Calculate the center of mass for each tower segment and check for balance
def check_centers_of_mass(tower: list, new_block: float, length=1, debug=False) -> bool:
    height = len(tower)
    msg = "{i} - Block center = {block}, Tower center = {com}, left bound = {lb}, right_bound = {rb}"
    if debug: print("\n--- Checking new block = {x} ---".format(x=new_block))
    for i, block in enumerate(tower):
        com = sum(tower[i:])/len(tower[i:])
        left_bound = block - length/2
        right_bound = block + length/2
        if debug: print(msg.format(i=i, block=block, com=com, lb=left_bound, rb=right_bound))
        if com < left_bound or com > right_bound:
            return False
    return True
```

Finally, we simulate the game of Jenga and keep adding blocks to the tower until it collapses. We run this simulation over 100,000 times in order to empirically obtain the average height of our tower before it falls down.

```
## Simulate the game of Jenga many many times to derive an empirical result
for _ in tqdm(range(iterations)):
    tower = [0]
    while True:
        new_block = generate_new_block(last_block=tower[-1], length=length)
        tower.append(new_block)
        success = check_centers_of_mass(tower, new_block, length)
        if not success:
            cumulative_height += len(tower)
            if len(tower) > h: h = len(tower)
            if len(tower) < l: l = len(tower)
            if debug: print("Jenga!! Final height was {n}".format(n=len(tower)))
            break
avg = cumulative_height/iterations
print(output_msg.format(avg=avg, h=h, l=l, ct=cumulative_height, i=iterations))
```
After running this simulation we can confidently assert that the solution to our riddle is 9.9 blocks.

```
100%|██████████| 100000/100000 [00:04<00:00, 22157.65it/s]
Simulation results:	Average=9.90683	Highest=58	Lowest=0	Cumulative Total=990683	Iterations=100000
```

# Riddler Express

https://fivethirtyeight.com/features/can-you-win-riddler-jenga/

This week marks the third of four CrossProduct™ puzzles. This time, there are seven three-digit numbers — each belongs in a row of the table below, with one digit per cell. The products of the three digits of each number are shown in the rightmost column. Meanwhile, the products of the digits in the hundreds, tens and ones places, respectively, are shown in the bottom row.

| 0 | 1 | 2 | Product |
| - | - | - | ------- |
|   |   |   | 280 |
|   |   |   | 168 |
|   |   |   | 162 |
|   |   |   | 360 |
|   |   |   | 60 |
|   |   |   | 256 |
|   |   |   | 126 |
|183,708|245,760|117,600|  |

Can you find all seven three-digit numbers and complete the table?

```
## Initialize the variables given in the problem
puzzle_rows = [280, 168, 162, 360, 60, 256, 126]
puzzle_columns = [183708, 245760, 117600]
```

This is a pretty fun puzzle that focuses on finding factors for each row and then testing each combination to find one that, when each column is multiplied, the product is equal to the number in the last row. It's similar to solving a sudoku puzzle where each cell of the table is constrained by a set of rules and we have to figure it out. I took a semi-brute-force approach because it's a pretty trivial problem and didn't want to spend too much time solving it. To help reduce the number of attempts that must be made from 10^21 (3 rows * 7 columns) down to 7 * 10^3 combinations to try when we check for column constraints. 

That's a pretty big difference and it shows! As a result of this work done to narrow down the list of feasible combinations we can solve this problem in under a second. We could optimize it further though this is an acceptable amount of work given that this problem is trivial. Next time that this puzzle pops up I'll try to solve it more intelligently by eliminating impossible combinations first.
```
## Identify every feasible combination for ever row
## Don't judge, this is quick and dirty
def build_row_combinations(rows: list) -> list:
    combinations = []
    for i, row in enumerate(rows):
        combinations.append([])
        for x in range(10):
            for y in range(10):
                for z in range(10):
                    if x*y*z == row:
                        combinations[i].append([x,y,z])
        print("Row {i} has {n} feasible combinations".format(i=i, n=len(combinations[i])))
    return combinations
```

The check_columns function then goes through every single combination, calculates the product of each column, and then compares it against the contraints in the last row.
```
## Check each combination to solve the problem
## Don't judge, this is quick and dirty
def check_columns(combinations: list, columns: list) -> list:
    for a in combinations[0]:
        for b in combinations[1]:
            for c in combinations[2]:
                for d in combinations[3]:
                    for e in combinations[4]:
                        for f in combinations[5]:
                            for g in combinations[6]:
                                result = True
                                for i, col in enumerate(columns):
                                    if a[i]*b[i]*c[i]*d[i]*e[i]*f[i]*g[i] != col:
                                        result = False
                                if result == True:
                                    print("Solution found!", a,b,c,d,e,f,g)
                                    return [a,b,c,d,e,f,g]
```
The last step is to solve the riddle!
```
## Solve the riddle
feasible_combos = build_row_combinations(puzzle_rows)
solution = check_columns(feasible_combos, puzzle_columns)
```
Here's the resulting solution!

# Solution
| A | B | C | Product |
| - | - | - | ------- |
| 7 | 8 | 5 | 280 |
| 3 | 8 | 7 | 168 |
| 9 | 6 | 3 | 162 |
| 9 | 8 | 5 | 360 |
| 3 | 5 | 4 | 60 |
| 4 | 8 | 8 | 256 |
| 9 | 2 | 7 | 126 |
|183,708|245,760|117,600|  |

| 0 | 1 | 2 | Product |
| - | - | - | ------- |
|   |   |   | 280 |
|   |   |   | 168 |
|   |   |   | 162 |
|   |   |   | 360 |
|   |   |   | 60 |
|   |   |   | 256 |
|   |   |   | 126 |
|183,708|245,760|117,600|  |
