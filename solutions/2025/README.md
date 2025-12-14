Advent of Code 2025 Solutions
=============================

This [year][aoc-2025] the Elves have good news and bad news.

The good news is that they've discovered project management! This has given them the tools they need to prevent their 
usual Christmas emergency. For example, they now know that the North Pole decorations need to be finished soon so that 
other critical tasks can start on time. The bad news is that they've realized they have a different emergency: according 
to their resource planning, none of them have any time left to decorate the North Pole!

To save Christmas, the Elves need us to finish decorating the North Pole by December 12th.

Table of Contents
-----------------

- [Highlights][hig]
- [Day 1 - Secret Entrance][d01]
- [Day 2 - Gift Shop][d02]
- [Day 3 - Lobby][d03]
- [Day 4 - Printing Department][d04]
- [Day 5 - Cafeteria][d05]
- [Day 6 - Trash Compactor][d06]
- [Day 7 - Laboratories][d07]
- [Day 8 - Playground][d08]
- [Day 9 - Movie Theater][d09]
- [Day 10 - Factory][d10]
- [Day 11 - Reactor][d11]

Highlights
----------

Each Advent of Code season comes with its own set of patterns, tricks, and small *aha* moments. Below is an overview
of some of the more interesting ideas I ended up implementing this year (e.g. specific algorithms). Many of these were
things I (re)discovered along the way, and writing them down here helps capture what made this year's puzzles especially
fun and sometimes even enlightening:

- ...

Day 1 - Secret Entrance
-----------------------
[Puzzle][d01-puzzle] — [Back to top][top]

We are given a document that contains a sequence of **rotations**, one per line, which will help
us open the safe. Each rotation consists of a **direction** and the number of **turns**.

```python
# Input:
document = open(...).read().splitlines()
```

### Part 1.1

Note that the dial has a range from 0 to 99 and we are only interested in the number of times the dial is left pointing
at 0 after any full rotation. Hence, we are only interested in the [modulo][mod-info] of 100 as any multiple of 100 
rotations will bring us back at our starting point:

```python
counter, dial = 0, 50
for rotation in document:
    direction, turns = rotation[0], int(rotation[1:])
    dial = (dial + turns if direction == 'R' else dial - turns) % 100
    if dial == 0:
        counter += 1
```

### Part 1.2

Now each individual turn of the dial within a given rotation should be checked, but this is easily done by simply adding
a for-loop to our previous solution:

```python
counter, dial = 0, 50
for rotation in document:
    direction, turns = rotation[0], int(rotation[1:])
    for _ in range(turns):
        dial = (dial + 1 if direction == 'R' else dial - 1) % 100
        if dial == 0:
            counter += 1
```

Day 2 - Gift Shop
-----------------
[Puzzle][d02-puzzle] — [Back to top][top]

We are given a few product **ID ranges** that we need to check to help out the clerks:

```python
# Input:
ids = open(...).read().split(",")
```

### Part 2.1

An invalid ID is defined by a sequence of digits **repeated twice**, therefore the problem actually boils down to simply 
checking if the first half of the string equals the second half of the string for each ID in the given range:

```python
total = 0
for ID in ids:
    start, end = list(map(int, ID.split('-')))
    for n in range(start, end + 1):
        middle = len(str(n))//2
        if str(n)[:middle] == str(n)[middle:]:
            total += n
```

### Part 2.2

Now an invalid ID is actually defined by a sequence of digits **repeated at least twice**.

Multiple solutions, why?! After solving part 2, I realised that I had tunnel vision and simply wanted to extend my
previous code to find the answer quickly. It worked, however, I realised afterward that it could be solved in a much easier manner.

#### Solution 2.2.1

The idea behind my first solution is along the same lines as the solution for part 1. Instead of only dividing each
sequence into two separate chunks to compare, I added a loop over the length of the sequence (half its length to be precise)
such that I can divide the sequence in different chunk sizes. Then we can simply check if all of them are the same:

```python
total = 0
for ID in ids:
    start, end = list(map(int,ID.split('-')))
    for n in range(start, end + 1):
        length = int(len(str(n)))
        for size in range(1, length//2 + 1):
            if length % size == 0:
                chunks = [str(n)[i:i+size] for i in range(0, len(str(n)), size)]
                if all(x == chunks[0] for x in chunks):
                    total += n
                    break
```

#### Solution 2.2.2

However, the previous solution is quite slow (5s) and that always gets me thinking during AoC: is there a trick such that
I can do less? In this particular case, is it possible to reduce the number of times we have to loop? The answer: **YES!**

```python
total = 0
for ID in ids:
    start, end = map(int, ID.split('-'))
    for n in range(start, end + 1):
        s = str(n)
        if len(s) > 1 and s in (s + s)[1:-1]:
            total += n
```

If a string `s` consists of a pattern `p` repeated `k` times (with `k>=2`), then `s = P + ... + p`. If we then
concatenate `s` with itself we get: `s + s = p + ... + p + p + ... + p`. If `s` has a repeating pattern, we
should be able to shift our view of `s + s` by one character and still have the pattern realign itself at some point before 
we reach the middle of the concatenated string. This means we do not actually need an additional loop at all, we just need
to concatenate the ID string and look within!

Day 3 - Lobby
-------------
[Puzzle][d03-puzzle] — [Back to top][top]

We are given a list of **joltage ratings** for the batteries of the broken escalator, where each line represents a 
**bank** of batteries:

```python
# Input:
ratings = open(...).read().splitlines()
```

### Part 3.1

Note that we are just asked to find the highest 2-digit number we can make from a string of numbers, while not violating 
their order within the original list. To avoid unnecessary loops we simply search the highest number among the first
`n-1` digits using the [max function][max-info] and then do the same for the remaining candidates:

```python
total = 0
for bank in ratings:
    first = max(bank[:-1])
    second = max(bank[bank.index(first)+1:])
    total += int(first + second)
```

### Part 3.2

The problem is now expanded to finding the highest 12-digit number, however we can still use the same [greedy approach][greedy-info]
as our first solution. Instead of finding one additional digit, we just loop and find 11 additional digits. At each
**step** our possible candidates are still all the remaining digits after the previous chosen joltage, but now we also
need to exclude all digits at the end that would prohibit us from ending up with 12 total digits:

```python
size, total = 12, 0
for bank in ratings:
    joltage = max(bank[:-size+1])
    for step in range(1, size):
        bank = bank[bank.index(joltage[-1])+1:]
        joltage += max(bank[:len(bank) - size + step + 1])
    total += int(joltage)
```

Day 4 - Printing Department
---------------------------
[Puzzle][d04-puzzle] — [Back to top][top]

What is Advent of Code without a **grid**-question and this year we are given the positioning of large **rolls** of paper
within the printing department of the elves:

```python
# Input:
grid = [list(row) for row in open(...).read().splitlines()]
```

### Part 4.1

To find the total number of rolls that can currently be accessed by the forklifts, we simply loop over the width (`X`)
and length (`Y`) of the grid and given a roll (`@`) count the adjacent rolls:

```python
rolls = 0
for x in range(X := len(grid[0])):
    for y in range(Y := len(grid)):
        if grid[x][y] == '@':
            count = 0
            for i in [-1,0,1]:
                for j in [-1,0,1]:
                        if 0<=(x+i)<X and 0<=(y+j)<Y and grid[x+i][y+j] == '@':
                            count += 1
            if count <= 4:
                rolls += 1
```
Remember to make sure we are still inside the grid when evaluating the adjacent spots, thus `0<=(x+i)<X` and `0<=(y+j)<Y` 
should hold!

### Part 4.2

The problem extension is to update the grid by removing the rolls of paper that can be accessed and to then continue
searching for new accessible rolls. This simply means we need to add a while-loop to our previous code and break if
no more additional rolls were moved:

```python
rolls = 0
while True:
    changed = False
    for x in range(X := len(grid[0])):
        for y in range(Y := len(grid)):
            if grid[x][y] == '@':
                count = 0
                for i in [-1,0,1]:
                    for j in [-1,0,1]:
                            if 0<=(x+i)<X and 0<=(y+j)<Y and grid[x+i][y+j] == '@':
                                count += 1
                if count <= 4:
                    rolls += 1
                    grid[x][y], changed = '.', True
    if not changed:
        break
```

Day 5 - Cafeteria
-----------------
[Puzzle][d05-puzzle] — [Back to top][top]

We are given a list of fresh ingredient **ID ranges** and a list of available **ingredient IDs** that we can use to help
the elves identify the spoiled ingredients:

```python
# Input:
ranges, ingredients = open(...).read().split("\n\n")
```

### Part 5.1

First we need to identify the amount of available fresh ingredients, which you can calculate by simply looping over the
available ingredients and evaluating if its ID is within one of the specified ranges:

```python
total = 0
R = [tuple(map(int,r.split('-'))) for r in ranges.splitlines()]
for ID in ingredients.splitlines():
    for (start, end) in R:
        if start <= int(ID) <= end:
            total += 1
            break
```


### Part 5.2

Now we are only interested in the number of total unique fresh ingredients. The number of IDs within a range is simply 
`end - start + 1`, however since the ranges can overlap we cannot just calculate this for each range and call it a day. 
The trick is to sort our list of ranges on their starting positions, as this allows us to sweep from left to right and 
only add new, previously unseen IDs to our total count:

```python
total, position = 0, 0
R = sorted([tuple(map(int,r.split('-'))) for r in ranges.splitlines()])
for (start, end) in R:
    if position >= start:
        start = position + 1
    if start <= end:
        total += end - start + 1
    position = max(position, end)
```

Day 6 - Trash Compactor
-----------------------
[Puzzle][d06-puzzle] — [Back to top][top]

While the older [cephalopods][pod-info] are trying to break us out of the garbage chute, we are given the **math homework** of the 
youngest one:

```python
# Input:
homework = open(...).read().splitlines()
```

### Part 6.1

Note that the individual **math problems** should actually be read as columns instead of the rows we retrieved.
Fortunately, we can simply transpose our view using the built-in [zip function][zip-info]:

|     | **BEFORE**                       | **AFTER**                          |
|-----|----------------------------------|------------------------------------|
| *0* | `['123', '328', '51', '64']`     | `('123', '45', '6', '*')`          |
| *1* |`['45', '64', '387', '23']`      | `('328', '64', '98', '*')`         |
| *2* |`['6', '98', '215', '314']`      | `('51', '387', '215', '+')`        |
| *3* |`['*', '*', '+', '+']`           | `('64', '23', '314', '+')`         |

This allows us to keep our solution short and elegant:

```python
problems, total = [line.split() for line in homework], 0
for problem in zip(*problems):
    total += eval(problem[-1].join(problem[:-1]))
```

### Part 6.2

Now the problem seems to become quite complex as the white spaces are actually important and the numbers should be 
read from top to bottom. However, when we use the same transpose as before on the raw strings we actually get an
interesting view:

|     | **LINES**         |
|-----|-------------------|
| *0* | `('1', ' ', ' ')` |
| *1* | `('2', '4', ' ')` |
| *2* | `('3', '5', '6')` |
| *3* | `(' ', ' ', ' ')` |
| *4* | `('3', '6', '9')` |

We can simply keep **track** of all the current numbers and when we encounter a 'white line' apply the corresponding
**operator**:


```python
total, tracked_numbers = 0, []
problems, operators = [line + ' ' for line in homework[:-1]], homework[-1].split()
for problem in zip(*problems):
    if all(n == ' ' for n in problem):
        total += eval(operators[0].join(tracked_numbers))
        tracked_numbers = []
        operators.pop(0)
    else:
        tracked_numbers.append("".join(x for x in problem if x.strip()))
```

Adding a space behind every string at the start is important as this creates an additional white line, otherwise the code
will ignore the last problem.

Day 7 - Laboratories
--------------------
[Puzzle][d07-puzzle] — [Back to top][top]

We are given a **diagram** of the tachyon manifolds, which should help us fix the teleportation device:

```python
# Input:
diagram = [list(line) for line in open(...).read().splitlines()]
```

### Part 7.1

The first part basically asks us to simulate the downward movements of the tachyon **beams** such that we can count the
total amount of **splits**. One approach is to use a [queue][q-info] to trace the movements:


```python
beams, splits = deque(), set()
beams.append((1, diagram[0].index('S')))
while beams:
    r, c = beams.popleft()
    if (0 <= r + 1 < len(diagram) and 0 <= c < len(diagram[0]) and (r+1,c) not in splits):
        if diagram[r + 1][c] == '.':
            beams.append((r+1,c))
        elif diagram[r + 1][c] == '^':
            splits.add((r+1, c))
            beams.append((r+1, c-1))
            beams.append((r+1, c+1))
```

### Part 7.2

Now we are actually interested in the number of unique **timelines** the tachyon can take if it can go either direction 
when it encounters a `^`. This is a classic example of a problem we can solve using a [backtracking algorithm][backtrack-info]:

```python
def backtrack(grid: list[list], r: int, c: int) -> int:
    if not (0 <= r < len(grid) and 0 <= c < len(grid[0])):
        return 0
    elif r == len(grid) - 1:
        return 1
    elif grid[r+1][c] == '.':
        return backtrack(grid, r+1, c)
    elif grid[r+1][c] == '^':
        return backtrack(grid, r+1, c-1) + backtrack(grid, r+1, c+1)
```

The above function works fine for small grids, but it actually uses a **naive recursion** approach instead of **dynamic**.
Why is this a problem? Because while you think you are traversing a tree, the problem is actually a [Directed Acyclic Graph (DAG)][dag-info], 
meaning it has overlapping sub-problems and therefore we are recomputing the entire future again from scratch for every timeline.

Fortunately, we can account for this via [memoization][memo-info] by caching known future positions and thus transforming
our time complexity from exponential `O(2^R)` to linear-*ish* `O(R·C)`:
```python
def backtrack(grid: list[list], r: int, c: int, cache: dict) -> int:
    if not (0 <= r < len(grid) and 0 <= c < len(grid[0])):
        return 0
    elif (r, c) in cache:
        return cache[(r, c)]
    elif r == len(grid) - 1:
        return 1
    elif grid[r+1][c] == '.':
        cache[(r+1,c)] = backtrack(grid, r+1, c, cache)
        return cache[(r+1,c)]
    elif grid[r+1][c] == '^':
        cache[(r+1,c)] = backtrack(grid, r+1, c-1, cache) + backtrack(grid, r+1, c+1, cache)
        return cache[(r+1,c)]
```

Allowing us to instantly find our answer by just plugging in the starting position:

```python
# Answer:
backtrack(grid=diagram, r=1, c=diagram[0].index('S'), cache={})
```

Day 8 - Playground
------------------
[Puzzle][d08-puzzle] — [Back to top][top]

We are given the Cartesian coordinates of all the electrical **junction boxes** which will help us aid the elves in
optimally connecting them:

```python
boxes = [tuple(map(int, line.split(','))) for line in open(...)]
```

Note that we need to calculate the [straight-line distance][dist-info] in three dimensions, which can be done as follows:

```python
def euclidean(p: tuple, q: tuple) -> float:
    return ((p[0] - q[0])**2 + (p[1] - q[1])**2 + (p[2] - q[2])**2)**0.5
```
Since only the relative magnitude is important and not the actual distance, we could skip the square-root.

### Part 8.1

We are asked to simulate the creation process of the **circuits**, starting with only the first 1.000 boxes. Unfortunately,
we cannot solve this without calculating the **distances** between all possible box combinations. However, we can make our
lives a lot easier by storing these distances in a sorted list. This allows us to just go through the list from top
to bottom and connect the circuits of the boxes in question (if possible):

```python
circuits = [[box] for box in boxes]
distance = sorted([(euclidean(b1,b2), b1, b2) for i, b1 in enumerate(boxes) for b2 in boxes[i+1:]])
for _ in range(1000):
    shortest_group = distance[0][1:]
    c1 = next(i for i, c in enumerate(circuits) if shortest_group[0] in c)
    c2 = next(i for i, c in enumerate(circuits) if shortest_group[1] in c)
    if c1 != c2:
        circuits[min(c1,c2)] += circuits.pop(max(c1,c2))
    distance.pop(0)
total = (lambda x: x[0] * x[1] * x[2])(sorted(map(len, circuits), reverse=True)[:3])
```

### Part 8.2

Now we are only interested in the **horizontal distance** (so x-coordinates) between the final two junction boxes. This 
means our code stays almost exactly the same, except now we just keep going until there is 1 circuit left:

```python
circuits = [[box] for box in boxes]
distance = sorted([(euclidean(b1,b2), b1, b2) for i, b1 in enumerate(boxes) for b2 in boxes[i+1:]])
while len(circuits) != 1:
    shortest_group = distance[0][1:]
    c1 = next(i for i, c in enumerate(circuits) if shortest_group[0] in c)
    c2 = next(i for i, c in enumerate(circuits) if shortest_group[1] in c)
    if c1 != c2:
        circuits[min(c1,c2)] += circuits.pop(max(c1,c2))
    distance.pop(0)
horizontal_distance = s[0][0] * s[1][0]
```

Day 9 - Movie Theater
---------------------
[Puzzle][d09-puzzle] — [Back to top][top]

We are given the list of (x, y) coordinates corresponding to the **red tiles** in the big grid:

```python
red_tiles = [tuple(map(int,line.split(','))) for line in open(...)]
```

### Part 9.1

To find the biggest possible rectangle using 2 red tiles (`p` & `q`) as its opposite corners, we can simply loop over 
all possible combinations and calculate the **area** of the square:

```python
area = 0
for i, p in enumerate(red_tiles):
    for q in red_tiles[i+1:]:
        area = max(area, (abs(p[0]-q[0])+1)*(abs(p[1]-q[1])+1))
```

### Part 9.2

So now we are informed that the red tiles are actually the corners of a [**polygon**][poly-info] and that we should only
consider rectangles that completely reside inside this polygon. Thus we can completely reuse our previous code, but just
add one extra check such that we only consider valid squares:

```python
area = 0
for i, p in enumerate(red_tiles):
    for q in red_tiles[i+1:]:
        if area_in_poly(p, q):
            area = max(area, (abs(p[0]-q[0])+1)*(abs(p[1]-q[1])+1))
```

Before we define the `area_in_poly()`, let us first initialize the **edges** of the polygon. We want to avoid saving all
the individual points that make up the edges as the numbers in our puzzle input are rather large. Therefore, I opt for a
normalized geometric representation of the edges, where I differentiate between horizontal and vertical edges and only 
store the end-points:

```python
corners, horizontal_edges, vertical_edges = red_tiles + [red_tiles[0]], [], []
for (x1, y1), (x2, y2) in zip(corners, corners[1:]):
    if x1 == x2:
        vertical_edges.append((x1, *sorted((y1, y2))))
    else:
        horizontal_edges.append((y1, *sorted((x1, x2))))
```

With that out of the way, let us now define how we check if the area of a rectangle is completely within the polygon.
Note that we should **NOT** consider this as a [point-in-polygon (PIP)][PIP-info] problem as the rectangles in question
are very large, hence ray tracing algorithms are off the table. Instead the geometric idea is that if a polygon passes 
through the interior of the rectangle, then part of the rectangle is outside, even if the center is inside. So we should
actually check if none of the edges are inside the polygon:

```python
def area_in_poly(p1: tuple[int, int], p2: tuple[int, int]) -> bool:
    # Initialize corners:
    (x_lo, x_hi), (y_lo, y_hi) = sorted([p1[0], p2[0]]), sorted([p1[1], p2[1]])
    
    # Evaluate vertical edges:
    for x, y1, y2 in vertical_edges:
        if x_lo < x < x_hi:
            if max(y_lo, y1) < min(y_hi, y2):
                return False
                
    # Evaluate horizontal edges:
    for y, x1, x2 in horizontal_edges:
        if y_lo < y < y_hi:
            if max(x_lo, x1) < min(x_hi, x2):
                return False 
    return True
```

Day 10 - Factory
----------------
[Puzzle][d10-puzzle] — [Back to top][top]

We are given the remaining parts of the manual for the **machines**, which includes indicator **light** diagrams,
**button** wiring schematics, and **joltage** requirements.

While the puzzle input is not structured very neatly, we can use the built-in python package for regular expression 
operations ([`re`][re-info]) to organize it:

```python
manual = ([],[],[])  # (lights, buttons, joltages)
for m in open(...).read().splitlines():
    manual[0].append(list(re.search(r"\[(.*?)\]", m).group(1)))
    manual[1].append([tuple(map(int,b.split(','))) for b in re.findall(r"\((.*?)\)", m)])
    manual[2].append(list(map(int,re.search(r"\{(.*?)\}", m).group(1).split(','))))
```

### Part 10.1

First they ask us to figure out the minimum amount of **buttons** you need to press to get the correct set of indicator **lights**
on for each machine. It is important to note here that pressing a button twice is useless, as it just undoes itself and
therefore cannot lead to the minimum. This makes the problem much simpler as we can just loop over a known group of 
possible combinations. Furthermore, we can simulate the press of a button by taking the difference of two sets:

```python
total = 0
for idx, (lights, buttons) in enumerate(zip(manual[0], manual[1])):
    goal, sols = set(idx for idx, x in enumerate(lights) if x == '#'), []
    for k in range(len(buttons) + 1):
        for combo in itertools.combinations(buttons, k):
            sol = set()
            for btn in combo:
                sol ^= set(btn)
            if sol == goal:
                sols.append(k)  
    total += min(sols)
```

Note that I used the built-in [`itertools`][iter-info] package of python to construct the set of possible button combinations.

### Part 10.2

Now we are asked to figure out the minimum amount of **buttons** you need to press to get the correct **joltage** levels 
for each machine. This also means that pressing a button twice is actually possible now, significantly increasing the 
complexity of this problem. 

My initial thought was, we can solve this as a [system of linear equations][sle-info]. For example, given one of the
example **manuals**:

```python
"[.##.] (3) (1,3) (2) (2,3) (0,2) (0,1) {3,5,4,7}"
```

We can rewrite the buttons and joltages as a linear system, then convert it to an [augmented matrix][aug-info] and 
finally reduce it to its [Row Echelon Form (REF)][REF-info] which should be solvable and have a minimum 
[Integer Linear Programming (ILP)][ILP-info] solution:

$$
\begin{aligned}
b_4 + b_5 &= 3 \\
b_1 + b_5 &= 5 \\
b_2 + b_3 + b_4 &= 4 \\
b_0 + b_1 + b_3 &= 7
\end{aligned}
\quad \Longrightarrow \quad
\left[
\begin{array}{ccccc|c}
0 & 0 & 0 & 1 & 1  & 3 \\
0 & 1 & 0 & 0 & 1  & 5 \\
0 & 0 & 1 & 1 & 0  & 4 \\
1 & 1 & 0 & 1 & 0  & 7
\end{array}
\right]
\quad \Longrightarrow \quad
\left[
\begin{array}{ccccc|c}
1 & 0 & 0 & 0 & -1 & 2 \\
0 & 1 & 0 & 0 & 1 & 5 \\
0 & 0 & 1 & 0 & -1 & 1 \\
0 & 0 & 0 & 1 & 1 & 3
\end{array}
\right]
$$

```python
total = 0
for buttons, joltages in zip(manual[1], manual[2]):
    # Create Augmented Matrix:
    n = len(joltages)
    A, b = [[int(i in btn) for btn in buttons] for i in range(n)], list(joltages)
    # Solve ILP:
    total += sum(solve(A,b))
```
Paragraph about `solve(A,b)`.

Paragraph about second approach (mention post).

```python
def get_combos(buttons: list[list[int]]) -> dict[tuple[int], int]:
    J, B, size = len(buttons[0]), len(buttons), {}
    for b in range(B + 1):
        for combo in itertools.combinations(buttons, b):
            action = (0,) * J if b == 0 else tuple(sum(c) for c in zip(*combo))
            size[action] = min(size.get(action, INF), b)
    return size
```

```python
def min_presses(buttons: [list[list[int]]], joltages: list[int]) -> int:
    combos = list(get_combos(buttons).items())

    @cache
    def solve(goal: tuple[int]) -> int:
        if not any(goal):
            return 0
        else:
            best = INF
            for combo, presses in combos:
                if all(c <= g and ((c ^ g) & 1) == 0 for c, g in zip(combo, goal)):
                    nxt = tuple((g - c) // 2 for c, g in zip(combo, goal))
                    best = min(best, presses + 2 * solve(nxt)) 
            return best

    return solve(tuple(joltages))
```

```python
total = 0
for (buttons, joltages) in zip(manual[1], manual[2]):
    btns = [[1 if i in btn else 0 for i in range(len(joltages))] for btn in buttons]
    total += min_presses(btns, joltages)
```

Day 11 - Reactor
----------------
[Puzzle][d11-puzzle] — [Back to top][top]

We are given a list of the **devices** in the server rack and their data connection to other devices in the rack:

```python
devices = {k: v.strip().split() for k, v in (l.split(":", 1) for l in open(...).read().splitlines())}
```

### Part 11.1

It is clear that we should interpret this as a graph question, where each device represents a **node** in the graph. 
Since data only ever flows from a device through its outputs, we can also conclude that we are dealing with another 
[DAG][dag-info] (so no loops). Long story short: *calculate the # of unique paths starting at node* `you` *and 
ending at node `out`*. Such a problem can be solved using a recursive implementation of [Depth-First Search (DFS)][DFS-info]:

```python
@cache
def n_paths(start, end):
    if start == end:
        return 1
    else:
        return sum(n_paths(node, end) for node in devices[start])
```

Note that I am again speeding up the process via [memoization][memo-info] by [caching][cache-info] paths in the graph
that we already walked.

### Part 11.2

After two days of quite complex second parts, we are actually getting a breather as we do not have to write any new code! 
They ask us for the number of unique paths from `svr` to `out` that also include the nodes `fft` and `dac`. But this is
simply equal to:

```python
n_paths('svr', 'fft') * n_paths('fft', 'dac') * n_paths('dac', 'out')
```

We did, however, had to add `out` to the `devices` object for this to work.

[aoc-2025]: https://adventofcode.com/2025

[top]: #advent-of-code-2025-solutions
[hig]: #highlights
[d01]: #day-1---secret-entrance
[d02]: #day-2---gift-shop
[d03]: #day-3---lobby
[d04]: #day-4---printing-department
[d05]: #day-5---cafeteria
[d06]: #day-6---trash-compactor
[d07]: #day-7---laboratories
[d08]: #day-8---playground
[d09]: #day-9---movie-theater
[d10]: #day-10---factory
[d11]: #day-11---reactor

[d01-puzzle]: https://adventofcode.com/2025/day/1
[d02-puzzle]: https://adventofcode.com/2025/day/2
[d03-puzzle]: https://adventofcode.com/2025/day/3
[d04-puzzle]: https://adventofcode.com/2025/day/4
[d05-puzzle]: https://adventofcode.com/2025/day/5
[d06-puzzle]: https://adventofcode.com/2025/day/6
[d07-puzzle]: https://adventofcode.com/2025/day/7
[d08-puzzle]: https://adventofcode.com/2025/day/8
[d09-puzzle]: https://adventofcode.com/2025/day/9
[d10-puzzle]: https://adventofcode.com/2025/day/10
[d11-puzzle]: https://adventofcode.com/2025/day/11

[mod-info]: https://en.wikipedia.org/wiki/Modulo
[max-info]: https://docs.python.org/3/library/functions.html#max
[zip-info]: https://docs.python.org/3/library/functions.html#zip
[q-info]: https://docs.python.org/3/library/collections.html#deque-objects
[re-info]: https://docs.python.org/3/library/re.html
[iter-info]: https://docs.python.org/3/library/itertools.html
[cache-info]: https://docs.python.org/3/library/functools.html


[greedy-info]: https://en.wikipedia.org/wiki/Greedy_algorithm
[backtrack-info]: https://en.wikipedia.org/wiki/Backtracking
[memo-info]: https://en.wikipedia.org/wiki/Memoization
[dag-info]: https://en.wikipedia.org/wiki/Directed_acyclic_graph
[dist-info]: https://en.wikipedia.org/wiki/Euclidean_distance
[poly-info]: https://en.wikipedia.org/wiki/Polygon
[PIP-info]: https://en.wikipedia.org/wiki/Point_in_polygon#Ray_casting_algorithm
[DFS-info]: https://en.wikipedia.org/wiki/Depth-first_search
[sle-info]: https://en.wikipedia.org/wiki/System_of_linear_equations
[aug-info]: https://en.wikipedia.org/wiki/Augmented_matrix
[REF-info]: https://en.wikipedia.org/wiki/Row_echelon_form
[ILP-info]: https://en.wikipedia.org/wiki/Integer_programming

[pod-info]: https://en.wikipedia.org/wiki/Cephalopod