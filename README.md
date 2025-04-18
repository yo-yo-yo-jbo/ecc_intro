# Introduction to Elliptic Curve Cryptography
This blogpost is going to be an introduction-level post about [Eliptic Curve Cryptography (ECC)](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography).  
It assumes knowledge in modular arithmetic as well as basic Group theory knowledge, both of which I have blogged about [in the past](https://github.com/yo-yo-yo-jbo/crypto_modular/).  

## What is an Elliptic Curve?
At the heart of elliptic curve cryptography lies a fascinating mathematical object: the elliptic curve. Unlike what the name might suggest, elliptic curves have nothing to do with ellipses. Instead, they're defined by a specific type of cubic equation and possess elegant algebraic properties that make them perfect for cryptography.

### The general equation
Over the real numbers, an elliptic curve is typically given by the *Weierstrass equation*:

$y^2=x^3+ax+b$

To ensure that the curve is smooth (i.e., it has no cusps or self-intersections), the *discriminant* must be nonzero:

$\Delta=−16(4a^3+27b^2)\neq0$

This condition ensures that the curve forms a well-behaved shape suitable for defining arithmetic.  
Here’s what an elliptic curve looks like when plotted over the real numbers with `a = -1` and `b = 2` (courtesy of [desmos.com](https://www.desmos.com)):  

![Elliptic Curve plot](ecc_plot.png)

Note the curve is symmetrical in relation to the `x` axis - this is because the only power of `y` in the equation is `2`.

### Point addition as a Group
Just like other curves, we could select two different points on the curve (let's call them `P` and `Q`) and connect them with a line.  
Amazingly, that line will hit that curve exactly *one* more time, unless the points have the same `x` coordinate.  
That quality gives us a strong motivation to build a mathematical `Group` out of it!  
As a reminder, a `Group` is a set of elements that have an `operation` defined on them. If we mark the operation as `+`, the requirements are:
- The operation is *closed* - for every two elements `x` and `y` the result `x+y` is also a part of the set.
- Identity element: the operation has a special element marked as `0` called *identity element* that satisfies `0+x = x+0 = x`.
- Associativity: every three elements `x`, `y` and `z` satisfy: `x+(y+z) = (x+y)+z`.
- Inverse element: for every element `x` there exists an element `y` that satisfies `x+y=0`, and we mark it as `y=-x`.

Note in my previous post about Group I marked things slightly differently - the identity was marked as `e` and the operation looked like multiplication rather than addition. However, those are just marks - you can think of them as [C++ operator overloading](https://en.cppreference.com/w/cpp/language/operators).  
In fact, from school, we got used to addition and multiplication to be *commutative*, so `x+y = y+x`. That is *not* necessarily true in Group theory, but in our case for `ECC` it will be true - we call commutative Groups [Abelian](https://en.wikipedia.org/wiki/Abelian_group).  
How far are we from saying that point addition on an Elliptic curve is a Group?  
Well, if the elements of our Group are the points on the curve, we know how to add two points to get a third, besides the following cases:
1. The points are the same point (i.e. we want to define `P+P`).
2. The points are not the same point but have the same `x` coordinate but oppositve `y` coordinates, i.e. `(x, y)` and `(x, -y)`.

Let us define those, and the entire Group!
- Because the curve is smooth ($\Delta\neq0$) each point has a tangent, and it crosses the line in exactly one more point, besides points that have `y=0` that create a perfectly vertical tangent, so besides those annoying points (there could be one or three of them), we already have `P+P` defined.
- If `P = (x, y)` it feels natural to define `-P = (x, -y)`. Connecting those two points with a line gets us nowhere... So let's define one more element in the Group! Geometrically, we won't see it, but we refer to is as *the point at Infinity* and mark it as $\mathcal{O}$. That will be our *identity element*.
- For 3 points `P`, `Q`, `R` on the same line, we define `P+Q+R` as $\mathcal{O}$, or, in other words: `P+Q = -R`. Geometrically, it means that we stretch a line between `P` and `Q`, get the third point and then take a the point "mirrored" as the result.

Here is how point addition looks like in a picture (again courtesy of [desmos.com](https://www.desmos.com)):

![Elliptic Curve plot](ecc_addition.png)

The "big" orange and blue dots represent the points `P` and `Q`, and the small *blue* point represents `P+Q`.
