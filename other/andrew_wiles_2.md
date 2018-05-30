# Challenge: Andrew Wiles 2

We're given a connection to some server where we must input three numbers `a`, `b`, and `c`, such that the server verifies whether they break Fermat's Last Theorem for `n = 3`, and if so, returns a flag.

We know from the description of the challenge and from its usage in the previous challenge Andrew Wiles that we can't just overflow an integer yet again.

There is another exploit that exists for forcing the sum of cubes to fail to be an injective function for unordered pair of integers - if we're working with floating points, whose possible values become more and more dispersed with larger and larger values, all we need is to find a near miss of Fermat's Last Theorem at `n = 3` with large values of `a^3`, `b^3`, and `c^3`.

An algorithm for this can be [found on StackExchange](https://math.stackexchange.com/a/847401/376050) by `Adam Bailey` that exploits the limiting behavior of a quotient of a monic and a cubic.

One example program that generates large enough numbers can be found below:

```c++
int main (int argc, char** argv)
{
    BigUnsigned m = 36627445161;

    std::cout << "a: " << (m * m * 3 + m * 2 + 1) << '\n';

    std::cout << "b: " << (m * 3 * m * m + m * 3 * m + m * 2) << '\n';

    std::cout << "c: " << (m * 3 * m * m +  m * 3 * m + m * 2 + 1) << '\n';
    return 0;
}
```