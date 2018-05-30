# Challenge: Masochist Keith

You're given a file of two columns of numbers - each row is a pair of coordinates for a point in space. You need to find the pair of points that gives the largest distance between them amongst all other possible pairs.

This problem is also known as finding the diameter of the convex hull of this set of points. The convex hull is a irregular polygon that whose inner angles are all smaller than `tau/2` and such that no other points lie outside it. It is easy to realize that both points of the largest-distance-pair must lie on that convex hull. Algorithms exist that can solve this problem in `O(log(n)) time` rather than the naive `O(n^2)`, making this problem solvable.

After reducing the set to the convex hull, you can use the rotating calipers method to find the largest-distance-pair amongst the points on the hull.