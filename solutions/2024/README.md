Advent of Code 2024 Solutions
=============================

This [year][aoc-2024] the Elves take us on a historical detour (celebrating the 10 year anniversary of AoC).

The Chief Historian, a familiar figure at Santa’s annual sleigh launch, has mysteriously vanished. The last anyone heard, 
he was traveling between locations of historical significance near the North Pole. Now, with Christmas approaching, a 
group of Senior Historians has enlisted our help to retrace his steps and piece together what happened.

To save Christmas, the elves need us to find the Chief Historian before Santa takes off on December 25th.

Table of Contents
-----------------

- [Highlights][hig]
- [Day 1 - Hystorian Hysteria][d01]
- [Day 2 - Red-Nosed Reports][d02]
- [Day 3 - Mull It Over][d03]
- [Day 4 - Ceres Search][d04]
- [Day 5 - Print Queue][d05]
- [Day 6 - Guard Gallivant][d06]
- [Day 7 - Bridge Repair][d07]
- [Day 8 - Resonant Collinearity][d08]
- [Day 9 - Disk Fragmenter][d09]
- [Day 10 - Hoof It][d10]
- [Day 11 - Plutonian Pebbles][d11]

Highlights
----------

Each Advent of Code season comes with its own set of patterns, tricks, and small *aha* moments. Below is an overview
of some of the more interesting ideas I ended up implementing this year (e.g. specific algorithms). Many of these were
things I (re)discovered along the way, and writing them down here helps capture what made this year's puzzles especially
fun and sometimes even enlightening:

- ...

Day 1 - Historian Hysteria
--------------------------
[Puzzle][d01-puzzle] — [Back to top][top]

We are given a document that contains two lists of **location IDs** side by side:

```python
# Input:
ids = [ID.split("   ") for ID in open(...).read().splitlines()]
```

### Part 1.1

For part one, the question is basically the answer itself as we are asked to pair up the IDs in ascending order and
calculate the distance between each pair:

```python
L, R = map(sorted, zip(*((int(l), int(r)) for l, r in ids)))
total = sum(abs(l - r) for l, r in zip(L, R))
```

### Part 1.2

Now we need to calculate a total **similarity score** by multiplying each number in the left list by the number of
occurrences it has in the right list:

```python
L, R = zip(*((int(l), int(r)) for l, r in ids))
score = sum(n * R.count(n) for n in L)
```

Day 2 - Red-Nosed Reports
-------------------------
[Puzzle][d02-puzzle] — [Back to top][top]

We are given many **reports**, each containing a list of safety **levels**:

```python
# Input:
reports = [[int(lvl) for lvl in line.split()] for line in open(...).read().splitlines()]
```

### Part 2.1

First we need to evaluate how many reports are safe based on the provided rules. So we simply loop over each
individual report `r`, check if consecutive levels (`a` & `b`) are not too far apart and if not check are the levels
ascending or descending:

```python
total = 0
for r in reports:
    if not all(1<=abs(a-b)<=3 for a,b in zip(r, r[1:])):
        continue
    elif all(a > b for a,b in zip(r, r[1:])) or all(a < b for a,b in zip(r, r[1:])):
        total += 1
```

### Part 2.2

Now unsafe reports are allowed if removing a single level creates a report that adheres to the rules. This means our code 
can stay almost exactly the same, we just need an additional loop such that we can evaluate all sub-report combinations `c`:

```python
total = 0
for r in reports:
    for c in (r[:i] + r[i+1:] for i in range(len(r))):
        if not all(1<=abs(a-b)<=3 for a,b in zip(c, c[1:])):
            continue
        elif all(a > b for a,b in zip(c, c[1:])) or all(a < b for a,b in zip(c, c[1:])):
            total += 1
            break
```

Day 3 - Mull It Over
--------------------
[Puzzle][d03-puzzle] — [Back to top][top]

We are given the corrupted **memory** of the shopkeepers' computer:

```python
# Input:
memory = open(...).read()
```

### Part 3.1

This is an obvious [regex][regex-info] problem for which we can use the built-in [`re`][re-info] module to find all the
valid multiplication strings that contain 1-3 digit numbers:

```python
sum(int(x)*int(y) for (x, y) in re.findall(r"mul\((\d{1,3}),(\d{1,3})\)", memory))
```

It is always a good idea to refresh your knowledge on regular expression syntax before starting with AoC, because such
puzzles tend to pop up every year.

### Part 3.2

Now we are only interested in valid multiplications that trail a `do()` statement and not a `don't()` statement. Since
my regex knowledge is not extremely deep, I just opted for a simple split:

```python
total = 0
for do in memory.split("do()"):
    for x, y in re.findall(r"mul\((\d{1,3}),(\d{1,3})\)", do.split("don't()")[0]):
        total += int(x) * int(y)
```

However, after a quick search, it would have also been possible to use regex to find the enabled blocks:

`r"(?s)(?:^|do\(\))(.*?)(?=don't\(\)|$)"`

Day 4 - Ceres Search
--------------------
[Puzzle][d04-puzzle] — [Back to top][top]

We are given the **grid** of the small elf's word search puzzle:

```python
# Input:
grid = open(...).read().splitlines()
```

### Part 4.1

Since the input is actually not that big, we can simply do a [brute-force][brute-info] approach to find all occurrences
of `"XMAS"` inside the puzzle. Just loop over the grid and at each position check all 4 important directions
(horizontal, vertical & two diagonals):

```python
total = 0
for r in range(R:=len(grid)):
    for c in range(C:=len(grid[0])):
        lines = [] 
        lines.append(c < C-3 and ''.join(grid[r][c+i] for i in range(4)))
        lines.append(r < R-3 and ''.join(grid[r+i][c] for i in range(4)))
        lines.append(c < C-3 and r < R-3 and ''.join(grid[r+i][c+i] for i in range(4)))
        lines.append(c > 2 and r < R-3 and ''.join(grid[r+i][c-i] for i in range(4)))
        total += sum(line in {"XMAS", "SAMX"} for line in lines)
```
Note that I also check for `"SAMX"` such that I do not have to evaluate 8 directions.

### Part 4.2

Apparently we are not accustomed to elfish puzzle logic, because we should have actually counted the number of times 
the word `"MAS"` creates a 3 by 3 cross inside the puzzle. We can use the same logic as before, but now at each position
we check the diagonals (`d1` & `d2`) of a 3x3 grid where our current position is the top left corner:

```python
total = 0
for r in range(R:=len(grid)):
    for c in range(C:=len(grid[0])):
        if (c < C - 2) and (r < R - 2):
            d1 = ''.join(grid[r+i][c+i] for i in range(3))
            d2 = ''.join(grid[r+i][c+2-i] for i in range(3))
            if d1 in {"MAS", "SAM"} and d2 in {"MAS", "SAM"}:
                total += 1
```

Day 5 - Print Queue
-------------------
[Puzzle][d05-puzzle] — [Back to top][top]

We are given an overview of the changes to be made to the new sleigh launch safety **manual**:

```python
# Input:
manual = open(...).read().split("\n\n")
```

It consists out of two parts, namely: the page ordering **rules** and the **update** specifications:

```python
rules = defaultdict(set)
for rule in manual[0].splitlines():
    first, second = map(int, rule.split('|'))
    rules[second].add(first)
```
```python
updates = [list(map(int, update.split(','))) for update in manual[1].splitlines()]
```
Note that I formatted the rules as a [defaultdict][ddict-info], where each page has a corresponding set of pages that
are not allowed to occur after the key page.

### Part 5.1

We are asked to evaluate which updates adhere to the given ruleset and count their total sum of middle pages. So we can simply
loop over the **pages** of each **update** and if we find an invalid sequence just continue to the next:
```python
total = 0
for update in updates:
    P = len(update)
    if any(set(update[p+1:]) & rules[update[p]] for p in range(P)):
        continue
    total += update[P//2]
```

### Part 5.2

Now we are actually interested in the invalid updates and ordering those correctly. Note that we can just add a `not`
operator in our previous *if-statement* to focus on invalid rules instead of valid rules! Furthermore, we can use a 
[queue][deque-info] to iteratively add pages to our revised update if the remaining pages in the queue do not violate
the rules:

```python
total = 0
for update in updates:
    P, new_update, queue = len(update), [], deque(update)
    if not any(set(update[p+1:]) & rules[update[p]] for p in range(P)):
       continue
    while queue:
        page = queue.popleft()
        if set(queue) & rules[page]:
            queue.append(page)
        else:
            new_update.append(page)
    total += new_update[P//2]
```

Day 6 - Guard Gallivant
-----------------------
[Puzzle][d06-puzzle] — [Back to top][top]

In order to prevent any time paradoxes, we are given a map of the **grid** that the guard is patrolling:

```python
# Input:
grid = open(...).read().splitlines()
```

### Part 6.1

We are asked to find the number of distinct positions that the guard visits before leaving the grid. To do this we first
need to find the **starting position** of the guard:

```python
start_pos = next((r, grid[r].index('^')) for r in range(len(grid)) if '^' in grid[r])
```
Next, we define a function that simulates the guard taking a step given her current `position` and `direction`:
```python
directions = cycle([(-1,0), (0,1), (1,0), (0,-1)])
def step(position: tuple, direction: tuple) -> tuple[tuple, tuple]:
    nxt = tuple(map(sum, zip(position, direction)))
    if grid[nxt[0]][nxt[1]] == '#':
        direction = next(directions)
        return step(position, direction)
    else:
        return (nxt, direction)
```
Note that I formatted the possible **directions** that the guard can take as a clockwise [cycle][cycle-info]. To acquire
all the distinct positions, we simply keep adding the **seen** positions to a set until we move outside the grid:

```python
pos, seen, d = start_pos, set({pos}), next(directions)
while (0 <= pos[0] < len(grid) - 1) and (0 <= pos[1] < len(grid[0]) - 1):
    pos, d = step(pos, d)
    seen.add(pos)
```

### Part 6.2

For the second part we are asked to find the number of unique loops we can create by adding one additional obstruction.
Unfortunately, I was not able to find a mathematical trick to identify the loops quickly. So I just used [brute-force][brute-info]
and looped over each position in the grid.

Since we are now running the simulation again for each new **obstruction**, I had to change the `step()` function slightly.
Besides adding the obstruction itself, I also started checking the range of the grid here and I realized that a cycle object
is actually not ideal (so I opted for a list `DIRS` instead):

```python
def step(position: tuple, direction: int, obstruction: tuple):
    nxt = tuple(map(sum, zip(position, DIRS[direction])))
    if not (0 <= nxt[0] < R and 0 <= nxt[1] < C):
        return None
    if (grid[nxt[0]][nxt[1]] == '#') or (nxt == obstruction):
        return (position, (direction + 1) % 4)
    return (nxt, direction)
```
Next, for each possible new obstruction (`obs`), we basically run exactly what we did in part one. However, now we also
store the corresponding direction of the seen positions. Because if we hit a (position, direction) combo that we already
encountered before, we know that we have found a loop:

```python
total = 0
for r in range(R:=len(grid)):
    for c in range(C:=len(grid[0])):
        if grid[r][c] != '.':
            continue
        else:
            pos, obs = start_pos, (r,c)
            d, seen = 0, set()
            while True:
                if (pos, d) in seen:
                    total += 1
                    break
                seen.add((pos, d))
    
                out = step(pos, d, obs)
                if out is None:
                    break
                pos, d = out
```

Day 7 - Bridge Repair
---------------------
[Puzzle][d07-puzzle] — [Back to top][top]

We are given the calibration **equations** of the defected bridge:

```python
# Input:
equations = [eq.split(": ") for eq in open(...).read().splitlines()]
```

### Part 7.1

First we need to identify which test **values** can be produced by placing any combination of **operators** (`+` or `*`)
into their corresponding equation. Note that instead of trying all possible combinations for each equation, we can also
interpret them as [binary trees][BT-info]. Since a binary tree is a [directed acyclic graph (DAG)][DAG-info] we can
identify **valid** equations via recursion using a [depth-first search (DFS)][DFS-info] algorithm:

```python
def valid(ans: int, nums: tuple[int], value: int):
    if len(nums) == 0:
        return ans == value
    return valid(ans + nums[0], nums[1:], value) or valid(ans * nums[0], nums[1:], value)
```
With that done, we only need to loop over the equations and evaluate if they are valid:
```python
total = 0
for value, numbers in equations:
    value, numbers = int(value), tuple(map(int, numbers.split()))
    if valid(numbers[0], numbers[1:], value):
        total += value
```

### Part 7.2

Now we are given a third operator (`||`) that literally adds the digits of two numbers together to create a new number.
It is quite easy to extend our previous code to account for this, as we only need to add the following operation
to the return statement of our `valid()` function:
```python
valid(int(str(ans)+str(nums[0])), nums[1:], value)
```

Day 8 - Resonant Collinearity
-----------------------------
[Puzzle][d08-puzzle] — [Back to top][top]

We are given a **grid** overview of the antenna locations:

```python
# Input:
grid = open(...).read().splitlines()
```

But we are actually more interested in the various antenna **frequencies** that are on the grid. So let us format our 
input like a dictionary where each **key** represents a frequency and the **element** is a list of all the corresponding
antenna locations:

```python
frequencies = defaultdict(list)
for r in range(R:=len(grid)):
    for c in range(C:=len(grid[0])):
        if (freq:=grid[r][c]) != '.':
            frequencies[freq].append((r,c))
```
Note that I used a [defaultdict][ddict-info] for convenience, as it initializes unseen frequencies with an empty list.

### Part 8.1

First we are asked to identify the number of unique **antinode** locations, where an antinode occurs at any point that
is perfectly in line with two antennas of the same frequency, but only when one of the antennas is twice as far away.

It took me some time to actually understand what this means (as I misread it multiple times). But mathematically speaking,
given two antennas of the same frequency, $a_1 = (r_1, c_1)$ and $a_2 = (r_2, c_2)$, we can define antinodes as:

$$
n_1 = a_2 + d \mbox{  and  } n_2 = a_1 - d \mbox{  with  } d = (r_2 - r_1, c_2 - c_1),
$$

where $d$ represents the displacement vector from $a_1$ to $a_2$. So for each frequency we should loop over all possible
antenna [combinations][combo-info], calculate their antinode locations and evaluate if they are inside the grid:

```python
nodes = set()
for freq in frequencies:
    for (r1, c1), (r2, c2) in combinations(frequencies[freq], 2):
        n1, n2 = (2*r2 - r1, 2*c2 - c1), (2*r1 - r2, 2*c1 - c2)
        for n in [n1, n2]:
            if 0 <= n[0] < R and 0 <= n[1] < C:
                nodes.add(n)
```

### Part 8.2

Of course, we forgot to take into the effects of *resonant harmonics* into our calculations. Antinodes actually occur at
any grid position exactly in line with at least two antennas of the same frequency, regardless of how far. So in
part 1 we solved for a **distance** of $k=1$, but now we just need to solve it for $k=1,...,R$:

```python
nodes = set()
for freq in frequencies:
    for (r1, c1), (r2, c2) in combinations(frequencies[freq], 2):
        for k in range(R):
            n1, n2 = (r2 + k*(r2-r1), c2 + k*(c2-c1)) , (r1 - k*(r2-r1), c1 - k*(c2-c1))
            for n in [n1, n2]:
                if 0 <= n[0] < R and 0 <= n[1] < C:
                    nodes.add(n)
```

Day 9 - Disk Fragmenter
-----------------------
[Puzzle][d09-puzzle] — [Back to top][top]

We are given the **disk map** of the amphipod's computer:

```python
# Input:
disk = open(...).read()
```

### Part 9.1

Before we can start, we need to be able to **decompress** the **disk**. For each **block** of 2 digits, we append the
decompressed **file system** with the number of saved **files** and the amount of free **storage**:

```python
def decompress(disk: str) -> list[list]:
    system, blocks = [], [disk[i:i+2] for i in range(0, len(disk),2)]
    for ID, (file, storage) in enumerate(blocks):
        for f in range(int(file)):
            system.append([ID])
        for s in range(int(storage)):
            system.append(None)
    return system
```

The goal of this problem is to **clean up** the storage space of the system. Be aware that this is not a sorting problem,
but a [two-pointer pattern][two-pointer-info]. By means of inward traversal we can start from both sides of the system
simultaneously (`L` & `R`) and move a file from **right** to **left** if possible:

```python
def clean(sys):
    L, R = 0, len(sys) - 1
    while L < R:
        if sys[L]:
            L += 1
        if not sys[R]:
            R -= 1
        if not sys[L] and sys[R]:
            sys[L] = sys[R]
            sys[R] = None
    return sys 
```

Given that we now can decompress and clean a file system, all that is left to do is calculate the total score:

```python
total = sum(i * b[0] for i, b in enumerate(clean(decompress(disk))) if b)
```

Note that I stored each block inside a list instead of keeping it all in a single string. This is because the ID numbers
in the actual puzzle input go beyond single digit numbers.

### Part 9.2

```python

```

Day 10 - Hoof It
----------------
[Puzzle][d10-puzzle] — [Back to top][top]

We are given a topographic **grid** of the area surrounding the lava production facility:

```python
# Input:
grid = [list(map(int, line)) for line in open(...).read().splitlines()]
```

### Part 10.1

The reindeer wants us to identify all *good hiking trails* and calculate their **scores**. A hiking trail is classified
as a path that starts at `0`, increases by 1 each step and ends at a **trailhead** `9`. The corresponding score is the
number of unique **heads** that can be reached from a single starting position.

Note that in order to **score** a hiking trail, given a starting point $(r,c)$, we are actually only interested in the
set of unique trailheads that can be reached from that point. Furthermore, we are dealing with a [DAG][DAG-info] again as 
one can only move forward within a *good hiking trail*. This allows us to speed things up by using a [recursive][recur-info] 
algorithm. In particular, another [DFS][DFS-info] approach, where from each position we explore all valid neighbors and
extend the set of encountered trailheads until we exhaust all possible paths:

```python
def score(r: int, c: int) -> set[tuple[int, int]]:
    heads = set()
    if (pos := grid[r][c]) == 9:
        return {(r,c)}
    else:
        for rr, cc in ((r, c+1), (r, c-1), (r+1, c), (r-1, c)):
            if 0 <= rr < R and 0 <= cc < C and grid[rr][cc] == pos + 1:
                heads |= score(rr, cc)
        return heads
```
Given our scoring function, we just have to add up the number of unique reachable trailheads for each starting position `0`:

```python
sum(len(score(r,c)) for r in range(len(grid)) for c in range(len(grid[0])) if grid[r][c] == 0)
```

### Part 10.2

This is a first, I accidentally solved part 2 before I solved part 1! We are now interested in the total **rating** of
a starting point, which is the number of distinct hiking trails. While trying to solve part 1, I initially forgot to 
track the trailheads that we already encountered. But that actually left me with the number of distinct hiking trails:

```python
def score(r: int, c: int) -> int:
    rating = 0
    if (pos := grid[r][c]) == 9:
        return 1
    else:
        for rr, cc in ((r, c+1), (r, c-1), (r+1, c), (r-1, c)):
            if 0 <= rr < R and 0 <= cc < C and grid[rr][cc] == pos + 1:
                rating += score(rr,cc)
        return rating
```

So now our scoring function just counts the doubles as well:

```python
sum(score(r,c) for r in range(len(grid)) for c in range(len(grid[0])) if grid[r][c] == 0)
```
Day 11 - Plutonian Pebbles
--------------------------
[Puzzle][d11-puzzle] — [Back to top][top]

We are given an overview of the current row of **stones** that we encountered on pluto:

```python
# Input:
stones = open(...).read().split()
```

### Part 11.1

Based on the rules with which the stones simultaneously change each time we **blink**, we need to simulate their behaviour
such that we can deduce the number of stones after a certain amount of blinking. Note that while they keep reemphasizing
that the order of the stones needs to be preserved, it is not important when solving this problem. Why? Because we can
just simulate what happens to each individual **stone** after $n$ blinks:

```python
def blink(stone: str, n: int) -> int:
    if n == 0:
        return 1
    elif stone == '0':
        return blink('1', n - 1)
    elif (L := len(stone)) % 2 == 0:
        a, b = str(int(stone[:L//2])), str(int(stone[L//2:]))
        return blink(a, n - 1) + blink(b, n - 1)
    else:
        return blink(str(int(stone) * 2024), n - 1)
```

This is a classic example of [dynamic programming][dp-info] on a compressed [state space][state-info]. Since they are 
interested in the amount of stones after 25 blinks, we can simply take the sum over each initial stone:

```python
sum(blink(stone, 25) for stone in stones)
```

### Part 11.2

Now they are interested in the amount of stones after 75 blinks, which is more computationally heavy for our function
from part one. However, due to the first two rules we will encounter a lot of states again. So applying [memoization][memo-info] 
via the built-in [cache][cache-info] decorator allows us to quickly calculate the answer for 75 blinks as well:

```python
sum(blink(stone, 75) for stone in stones)
```



[aoc-2024]: https://adventofcode.com/2024
[top]: #advent-of-code-2024-solutions
[hig]: #highlights
[d01]: #day-1---historian-hysteria
[d02]: #day-2---red-nosed-reports
[d03]: #day-3---mull-it-over
[d04]: #day-4---ceres-search
[d05]: #day-5---print-queue
[d06]: #day-6---guard-gallivant
[d07]: #day-7---bridge-repair
[d08]: #day-8---resonant-collinearity
[d09]: #day-9---disk-fragmenter
[d10]: #day-10---hoof-it
[d11]: #day-11---plutonian-pebbles

[d01-puzzle]: https://adventofcode.com/2024/day/1
[d02-puzzle]: https://adventofcode.com/2024/day/2
[d03-puzzle]: https://adventofcode.com/2024/day/3
[d04-puzzle]: https://adventofcode.com/2024/day/4
[d05-puzzle]: https://adventofcode.com/2024/day/5
[d06-puzzle]: https://adventofcode.com/2024/day/6
[d07-puzzle]: https://adventofcode.com/2024/day/7
[d08-puzzle]: https://adventofcode.com/2024/day/8
[d09-puzzle]: https://adventofcode.com/2024/day/9
[d10-puzzle]: https://adventofcode.com/2024/day/10
[d11-puzzle]: https://adventofcode.com/2024/day/11
[d12-puzzle]: https://adventofcode.com/2024/day/12
[d13-puzzle]: https://adventofcode.com/2024/day/13
[d14-puzzle]: https://adventofcode.com/2024/day/14
[d15-puzzle]: https://adventofcode.com/2024/day/15
[d16-puzzle]: https://adventofcode.com/2024/day/16
[d17-puzzle]: https://adventofcode.com/2024/day/17
[d18-puzzle]: https://adventofcode.com/2024/day/18
[d19-puzzle]: https://adventofcode.com/2024/day/19
[d20-puzzle]: https://adventofcode.com/2024/day/20
[d21-puzzle]: https://adventofcode.com/2024/day/21
[d22-puzzle]: https://adventofcode.com/2024/day/22
[d23-puzzle]: https://adventofcode.com/2024/day/23
[d24-puzzle]: https://adventofcode.com/2024/day/24
[d25-puzzle]: https://adventofcode.com/2024/day/25

[regex-info]: https://en.wikipedia.org/wiki/Regular_expression
[brute-info]: https://en.wikipedia.org/wiki/Brute-force_search
[BT-info]: https://en.wikipedia.org/wiki/Binary_tree
[DAG-info]: https://en.wikipedia.org/wiki/Directed_acyclic_graph
[DFS-info]: https://en.wikipedia.org/wiki/Depth-first_search
[two-pointer-info]: https://bytebytego.com/courses/coding-patterns/two-pointers/introduction-to-two-pointers?fpr=javarevisited
[recur-info]: https://en.wikipedia.org/wiki/Recursion_(computer_science)
[memo-info]: https://en.wikipedia.org/wiki/Memoization
[dp-info]: https://en.wikipedia.org/wiki/Dynamic_programming
[state-info]: https://en.wikipedia.org/wiki/State_space_(computer_science)

[re-info]: https://docs.python.org/3/library/re.html
[ddict-info]: https://docs.python.org/3/library/collections.html#collections.defaultdict
[deque-info]: https://docs.python.org/3/library/collections.html#collections.deque
[cycle-info]: https://docs.python.org/3/library/itertools.html#itertools.cycle
[combo-info]: https://docs.python.org/3/library/itertools.html#itertools.combinations
[cache-info]: https://docs.python.org/3/library/functools.html