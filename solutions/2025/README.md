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

Signature Moves
---------------
Each year the puzzles seem to have something different in common in terms of steps needed to find the solution. Often
these steps can easily be performed using built-in python features and therefore AoC serves as a good refresher of methods
you tend to forget about. Below is an overview of methods/ideas that define 2025s signature:

- [Modulo Operator][mod-info]: The use of mod (`%`) to quickly receive the signed remainder of a division.

Day 1 - Secret Entrance
-----------------------
[Puzzle][d01-puzzle] — [Back to top][top]

We are given a document that contains a sequence of **rotations**, one per line, which will help
us open the safe.

```python
# Input:
document = open(...).read().split("\n")
```

### Part 1

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

### Part 2

Now each individual turn of the dial within a given rotation should be checked, but this is easily done by simply adding
an additional for-loop to our previous solution:

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

### Part 1

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

### Part 2

Now an invalid ID is actually defined by a sequence of digits **repeated at least twice**.

Multiple solutions, why?! After solving part 2, I realised that I had tunnel vision and simply wanted to extend my
previous code to find the answer quickly. It worked, however, I realised afterward that it could be solved in a much easier manner.

#### Solution 1

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

#### Solution 2

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

[aoc-2025]: https://adventofcode.com/2025

[top]: #advent-of-code-2025-solutions
[sig]: #signature-moves
[d01]: #day-1---secret-entrance
[d02]: #day-2---gift-shop

[d01-puzzle]: https://adventofcode.com/2025/day/1
[d02-puzzle]: https://adventofcode.com/2025/day/2

[mod-info]: https://en.wikipedia.org/wiki/Modulo