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
`r"(?s)(?:^|do\(\))(.*?)(?=don't\(\)|$)"`.

[aoc-2024]: https://adventofcode.com/2024
[top]: #advent-of-code-2024-solutions
[hig]: #highlights
[d01]: #day-1---historian-hysteria
[d02]: #day-2---red-nosed-reports
[d03]: #day-3---mull-it-over


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

[re-info]: https://docs.python.org/3/library/re.html