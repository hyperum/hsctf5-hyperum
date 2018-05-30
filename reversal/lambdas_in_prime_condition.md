# Challenge: Lambdas in Prime Condition

You've got a file of lambdas within lambdas within lambdas. What now?

First, refactor the lambdas into functions, either manually or with a tool such as PyCharm.

You will get something like this: (WARNING: all code from now on should be treated as pseudocode, showing intentions, but not necessarily a well formed program.)

```python
# NB: ignore recursion limit
import sys
sys.setrecursionlimit(1500)

def call_self(canary):
    return canary(canary)

def perfidy(n):
    def parity(singapore):
        return lambda elegy, clerk: singapore(elegy[1:], clerk - 1) if clerk * len(elegy) - clerk else elegy[0]

    def sanctity(a):
        return lambda b, i: (i * i > b or b % i != 0) and a(b, i + 1)

    def entity(a):
        return call_self(lambda b: a(lambda *c: b(b)(*c)))

    def solidarity(x):
        return x * x - 2 * x + 2

    def corollary(ontology):
        return lambda formic: entity(parity)((solidarity,) * 2 + (lambda y: ontology(y - 1) + ontology(y - 2),), formic)(
            formic)

    def certainty(f):
        return lambda a, b, i: i if a == b else (b <= a and f(a, entity(corollary)(i + 1), i + 1))
    return entity(sanctity)(n, 2) * (entity(certainty)(n, 2, 0) != False)

anaconda = lambda a: perfidy(10000) if 10000 == a else (False if perfidy(10000) else (lambda d: lambda k: perfidy(k) if k == a else (False if perfidy(k) else d(d)(k + 1)))
            (lambda f: lambda m: perfidy(m) if m == a else (False if perfidy(m) else f(f)(m + 1)))(10001))

(lambda k: print("you found it:", k) if anaconda(perfidy)(k) else print("try again"))(int(input()))
```

Notice anything strange?

```python
def entity(a):
    return call_self(lambda b: a(lambda *c: b(b)(*c)))
```

This appears to be infinite recursion.

But closer inspection reveals it to be [something more reasonable](en.wikipedia.org/wiki/Fixed-point_combinator).

With this insight, let's substitute all calls of entity.

From then on, it is trivial to deduce that the program is looking for the first number above 10,000 that is prime and a member of the Lucas sequence, a sequence similar to that of the Fibonacci sequence but with the first two numbers 2, 1. That number happens to be 3,010,349, and that is the solution to the problem.

The resulting code is annotated for the reader's convenience.
```python
def call_self(canary):
    return canary(canary)

def parity(elegy, clerk): 
    return parity(elegy[1:], clerk - 1) if clerk * len(elegy) - clerk else elegy[0]
    # returns elegy[0] when clerk is 0, we're taking slices of the tuple.

def sanctity(b, i):
    return (i * i > b or b % i != 0) and sanctity(b, i + 1)
    # b is prime.

def solidarity(x):
    return x * x - 2 * x + 2

def corollary(formic):
    return parity((solidarity,) * 2 + (corollary(formic - 1) + corollary(formic - 2),), formic) 
    # looks like a fibonacci integer sequence, is that addition or tuple concatenation?

def certainty(a, b, i):
    return i if a == b else (b <= a and certainty(a, corollary(i + 1), i + 1))
    # a is an element of a fibonacci sequence starting with b, i + 1?

def perfidy(n):
    return sanctity(n, 2) * (certainty(n, 2, 0))
    # it appears as though we want a prime that is a Lucas number (F_0 = 2, F_1 = 1, ...)

def anaconda(a):
    return False if 10000 == a 
    # first solution after 10000
    else outer:(perfidy(10001) if 10001 == a else (False if perfidy(k) else outer(outer)(k + 1)))

(lambda k: print("you found it:", k) if anaconda(perfidy)(k) else print("try again"))(int(input()))
```