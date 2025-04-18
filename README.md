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
Amazingly, that line will hit that curve exactly *one* more time, unless the points have the same `x` value.  
That quality gives us a strong motivation to build a mathematical `Group` out of it!  
As a reminder, a `Group` is a set of elements that have an `operation` defined on them. If we mark the operation as `+`, the requirements are:
- The operation is *closed* - for every two elements `x` and `y` the result `x+y` is also a part of the set.
- Identity element: the operation has a special element marked as `0` called *identity element* that satisfies `0+x = x+0 = x`.
- Associativity: every three elements `x`, `y` and `z` satisfy: `x+(y+z) = (x+y)+z`.
- Inverse element: for every element `x` there exists an element `y` that satisfies `x+y=0`, and we mark it as `y=-x`.
Note in my previous post about Group I marked things slightly differently - the identity was marked as `e` and the operation looked like multiplication rather than addition. However, those are just marks - you can think of them as [C++ operator overloading](https://en.cppreference.com/w/cpp/language/operators).  
In fact, from school, we got used to addition and multiplication to be *commutative*, so `x+y = y+x`. That is *not* necessarily true in Group theory, but in our case for `ECC` it will be true - we call commutative Groups [Abelian](https://en.wikipedia.org/wiki/Abelian_group).  
How far are we from saying that point addition on an Elliptic curve is a Group?

