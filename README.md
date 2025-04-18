# Introduction to Elliptic Curve Cryptography
This blogpost is going to be an introduction-level post about [Eliptic Curve Cryptography (ECC)](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography).  
It assumes knowledge in modular arithmetic as well as basic Group theory knowledge, both of which I have blogged about [in the past](https://github.com/yo-yo-yo-jbo/crypto_modular/).  
I will also be referring several times to my [Diffie-Hellman key exchange blogpost](https://github.com/yo-yo-yo-jbo/dh_key_exchange/).

## What is an Elliptic Curve?
At the heart of Elliptic Curve Cryptography lies a fascinating mathematical object: the elliptic curve. Unlike what the name might suggest, elliptic curves have nothing to do with ellipses. Instead, they're defined by a specific type of cubic equation and possess elegant algebraic properties that make them perfect for cryptography.  
One great thing about `ECC` is its small key size requirements - a `256` bit key on an Elliptic Curve is as good as a `4096` RSA private key.

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
- Points that have a `y` coordinate of `0` (as I mentined, there could be one or three of them) are the inverse of themselves, so for those points `P = -P`, which also tells you `P+P` is just $\mathcal{O}$.

Here is how point addition looks like geometrically (again courtesy of [desmos.com](https://www.desmos.com)):

![Elliptic Curve plot](ecc_addition.png)

The "big" orange and blue dots represent the points `P` and `Q`, and the small *blue* point represents `P+Q`.  
Note how the Group we defined is also *Abelian* (as well as passes all requirements to be a mathematical Group).

### Using Elliptic Curves for cryptography
So, we built a group - that's nice and all, but how is it practical?  
For cryptography and computers in general, we are usually interested in *countable* numbers. I won't go into the details about what that means exactly, but in our case I'd say we simply have too many points (currently - [real numbers](https://en.wikipedia.org/wiki/Real_number)).  
Additionally, just like we saw in my [Diffie-Hellman blogpost](https://github.com/yo-yo-yo-jbo/dh_key_exchange/), things get interesting when you work `mod` some number (we used a prime in Diffie-Hellman).  
In `ECC`, we will either use a prime number or a large power of 2, but let's stick with primes for now. Thus, we will work with all integer numbers satisfying:

$y^2=x^3+ax+b (mod p)$

What different does it make?  
First of all, the number of points is finite, but most importantly - addition is done `mod p`, and therefore looks unpredictable, just like raising a number to large powers `mod p` looked unpredictable in Diffie-Hellman.  
Let's continue discussing the addition of a point to itself - we have defined `P+P`, and we can do that over and over and get new points.  
Our mathematical notation will be `P+P = 2P` and generally we could get `P+P+P+...+P = nP`. We call these *coefficient multiplication* or *scalar multiplication* of a point.  
Interestingly, that scalar multiplication is what we call a *trapdoor function* - it's relatively easy to calculate but hard to invert, and behaves very much like the [Discrete Logarithm](https://en.wikipedia.org/wiki/Discrete_logarithm) I mentioned in my [Diffie-Hellman key exchange blogpost](https://github.com/yo-yo-yo-jbo/dh_key_exchange/).  
Formally, if we have a *base point* marked as `P` and a very large positive integer `n`, it's hard to get `n` from the points `P` and `nP`.  
In a sense, you could think of the pair `(P, nP)` as a *public key* and `n` as a *private key*.

### Elliptic Curve Diffie-Hellman
Similarly to [Diffie-Hellman key exchange protocol](https://github.com/yo-yo-yo-jbo/dh_key_exchange/), we an define a similar protocol over the Group we have defined.  
Let's assume `Alice` and `Bob` agree on the curve, the prime `p` and a base point `P`. Then:
1. `Alice` chooses a random large number `a` and sends `Bob` the point `aP`.
2. `Bob` chooses a random large number `b` and sends `Alice` the point `bP`.
3. `Alice` gets `Bob`'s point and multiplies it with her `a` scalar, getting `a(bP) = abP`.
4. Similarly, `Bob` gets `Alice`'s point and multiplies it with his `b` scalar, getting `b(aP) = abP`.
5. Now `Alice` and `Bob` have exchanged a secret `abP` - for example, they could use the `x` coordinate of that `abP` point.
6. Note an evesdropper is not capable of concluding `a` from `aP` or `b` from `bP`.

