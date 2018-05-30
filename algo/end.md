# Challenge: End

You're given a file representing a matrix of 1s and 0s, such that you must start from the bottom left, move to the bottom right, moving only up and right, while not crossing any 1s. How many solutions are there?

Well, you can't just recurse up to a massive depth to find the solutions one-by-one. Instead, you can use dynamic programming to reconsider the problem more strategically.

First, notice that for each 0 tile, the number of solutions ending at that tile are equal to the number of solutions ending at the tile's immediate neighbors below and to the left.

Furthermore, the flag is modulo 10^9 + 7, so all your calculations can be done modulo this number so that you don't take up too much space for each element in the 5000 by 5000 matrix.

A solution-counting program is shown below:

```python
f = open("end.txt")
lines = f.read().splitlines()

lr = len(lines)
lc = len(lines[0].split())

limit = 1000000007

grid = [] # contents of file

for row in lines:
    text = []
    for col in row.split():
        text.append(int(col))
    grid.append(text)

solutions = [[0 for r in range(lc)] for c in range(lr)] # solutions per tile
solutions[lr - 1][0] = 1

for i in reversed(range(0, lr)):
    for j in range(0, lc):
        if grid[i][j] == 0:
            if i < lr - 1 and j > 0:
                solutions[i][j] = (solutions[i + 1][j] + solutions[i][j - 1]) % limit
            elif i < lr - 1:
                solutions[i][j] = solutions[i + 1][j]
            elif j > 0:
                solutions[i][j] = solutions[i][j - 1]

print "Number of solutions for the top-right tile:"
print solutions[0][lc - 1]
```
