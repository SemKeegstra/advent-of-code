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

- [Signature Moves][sig]
- [Day 1 - Secret Entrance][d01]
- [Day 2 - Gift Shop][d02]
- [Day 3 - Lobby][d03]
- [Day 4 - Printing Department][d04]

Signature Moves
---------------
Each year the puzzles seem to have something different in common in terms of steps needed to find the solution. Often
these steps can easily be performed using built-in python features and therefore AoC serves as a good refresher of methods
you tend to forget about. Below is an overview of methods/ideas that define 2025s signature:

- **Division Components**: The use of the modulo operator (`%`) and/or floor division (`//`) to quickly receive and/or remove
                           the rest terms of a division.

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
                        if 0<=(x+i)<X and 0<=(y + j)<Y and grid[x+i][y+j] == '@':
                            count += 1
            if count <= 4:
                rolls += 1
```
Remember to make sure we are still inside the grid when evaluating the adjacent spots, thus `0<=(x+i)<X` and `0<=(y + j)<Y` 
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
                            if 0<=(x+i)<X and 0<=(y + j)<Y and grid[x+i][y+j] == '@':
                                count += 1
                if count <= 4:
                    rolls += 1
                    grid[x][y], changed = '.', True
    if not changed:
        break
```

[aoc-2025]: https://adventofcode.com/2025

[top]: #advent-of-code-2025-solutions
[sig]: #signature-moves
[d01]: #day-1---secret-entrance
[d02]: #day-2---gift-shop
[d03]: #day-3---lobby
[d04]: #day-4---printing-department

[d01-puzzle]: https://adventofcode.com/2025/day/1
[d02-puzzle]: https://adventofcode.com/2025/day/2
[d03-puzzle]: https://adventofcode.com/2025/day/3
[d04-puzzle]: https://adventofcode.com/2025/day/4

[mod-info]: https://en.wikipedia.org/wiki/Modulo
[max-info]: https://docs.python.org/3/library/functions.html#max
[greedy-info]: https://en.wikipedia.org/wiki/Greedy_algorithm