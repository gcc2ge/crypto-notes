# Pairing-based cryptography

The following notation will be used:

```
k~: an algebraic closure of k
```

## Bilinear pairing or bilinear map

Notes mostly taken from [1] and [2].

Let G_1, G_2 be two cyclic groups of the same prime order n. Let us view G_1 as an additive group with identity ∞ and G_2 as a multiplicative group with identity 1. Let P be a generator of G_1.

A mapping e: G_1 x G_1 -> G_2 is called bilinear map when:

 * e(aP, bQ) = e(P, Q)^(ab) for all P, Q from G_1 and a, b from Z_q* (bilinearity)
 * e(P, P) != 1, which means if P is a generator of G_1, e(P, P) is a generator of G_2 (non-degeneracy)
 * e can be efficiently computed (computability)

For all S, T from G_1 it holds:

 * e(S, ∞) = 1 and e(∞ ,S) = 1 
 * e(S, -T) = e(-S, T) = e(S, T)^(-1)
 * e(aS, bT) = e(S, T)^(ab) for all a, b from Z
 * e(S, T) = e(T, S)
 * if e(S, R) = 1 for all R from G_1, then S = ∞. 

A symmetric bilinear map is completely defined by the value it takes at e(P, P). There are n bilinear maps for G1, one of them is degenerate case. The other n-1 are equivalent up to a constant (e_1(S, T) = e_2(S, T)^c for all S, T from G1). What is difficult is to find a map that is efficiently computable.

## Discrete logarithm problem (DLP)

If we have an additively-written group G = <P> of order n and elements P and Q, DLP is the problem to find the integer x from [0, n-1] such that Q = xP. DLP is for example believed to be intractable in multiplicative group of a finite field and in group of points on elliptic curve over a finite field.

Due to the bilinearity property, DLP in G_1 can be efficiently reduced to the DLP in G_2.

This is because if we have (P, Q) in G_1 where Q = xP, then e(P, Q) = e(P, xP) = e(P, P)^x.

## Diffie-Hellman problem (DHP)

DHP is closely related to DLP. The problem is: given P, aP and bP, find abP.

The security of Diffie-Hellman key agreement protocol relies on DHP.

## Decisional Diffie-Hellman problem in cyclic groups with bilinear maps

Given P, aP, bP, cP, find if ab = c. As e(aP, bP) = e(P, P)^(ab), we can simply check whether this value is the same as e(P, P)^c. 

Thus, the cryptosystems relying on intractability of decisional Diffie-Hellman cannot be constructed in a cyclic group with bilinear pairing.

Note that if e(P, P) = 1 for all P, the above check does not work. In this case the check can be done using distortion maps.

## Bilinear Diffie-Hellman problem (BDHP)

The security of many pairing-based protocols relies on the intractabilty of BDHP which is as follows: given P, aP, bP, cP, compute e(P, P)^(abc).

Hardnes of BDHP implies the hardness of DHP in both G_1 and G_2. The problem is generally assumed to be as hard as DHP in G_1 and G_2.

## Decisional Bilinear Diffie-Hellman Problem

Given P, aP, bP, cP, e(P, P)^w, determine whether w = abc.

## Some pairing-based protocols

### Three-party one-round key agreement

How can a key be exchanged between three parties in one round? Using Diffie-Hellman there are two rounds needed for three parties.

Using bilinear pairing, each of the three parties randomly selects a secret integer from [1, n-1] and broadcast the point to the other two parties (say Alice chooses a and broadcasts aP, Bob has b and broadcasts bP, Chris has c and broadcasts cP). Now each one of them can calculate a shared secret: Alice calculates it as e(bP, cP)^a, Bob as e(aP, cP)^b, Chris as e(aP, bP)^c. The shared secret is e(P, P)^(abc).

For parties that do not know either a, b, or c, the shared secret can be computed only as solving BDHP.

### Short signatures

Boneh, Lynn and Shacham (BLS) signature is created as follows:

 * Alice randomly chooses a from [1, n-1] which is her private key, A = aP is public key
 * H is hash function from {0, 1}* to G1\{∞}
 * signature on message m is created as S = aM where M = H(m)

The verifier just needs to check whether e(P, aM) = e(aP, M) (verifier knows S = aM, A = aP).

### Identity-based encryption

In identity-based cryptography a public key of Alice consists of her identifying information ID_A, like her e-mail address. A Trusted Third Party (TTP) use its private key to generate Alice'\'s private key from ID_A and securely transmit it to Alice.

Bob can encrypt messages for Alice using only ID_a and the TTP public key (Bob can encrypt message even before the private key for Alice is generated).

Such a system can be achived using scheme proposed by Boneh and Franklin:

 * we have bilinear pairing e for which BDHP is intractable
 * we have two hash functions H1: {0, 1}* -> G1\{∞} and H2: G_2 -> {0, 1}^l, where l is the bitlength of the plaintext
 * TTP private key is a randomly selected integer t from [1, n-1], its public key is T = tP
 * when Alice requests her private key d_A, TTP creates Alice''s identity string ID_A, computes d_A = tH_1(ID_A) and securely delivers d_A to Alice
 * Bob encrypts the message m from {0, 1}^l as Q_A = H1(ID_A), selects a random integer r from [1, n-1], computes R = rP, and finally computes c = m xor H2(e(Q_a, T)^r) and trnasmits the ciphertext (R, c) to Alice
 * Alice decrypts m = c xor H2(e(d_A, R)) (this works because e(d_A, R) = e(tQ_A, rP) = e(Q_A, tP)^r = e(Q_A, T)^r

Note that this scheme is secure against eavesdroppers, but is not resistant to chosen-ciphertext attacks. Let us say we have a ciphertext (R, c) and can somehow obtain a decryption m1 for (R, c1) where c1 is the same as c, only the first bit is flipped. Now we know m is the same as m1, only the first flip needs to be flipped. However, resistance to chosen ciphertext attacks can be achieved by using two additional hash functions which we will not cover here.

## Extension of pairing definition

Symmetric pairings can be instantiated by using suitable supersingular elliptic curves. To allow a wider range of curves to be used, the definition must be modified. Boneh, Lynn and Shacham used in [3] the following definition (multiplicative notation is used here).

Let G1, G2, G3 be cyclic groups of prime order r. Assume the DHP is hard in G1. Let phi: G2 -> G1 be an efficiently computable group isomorphism. Let g2 be a generator of G2 and g1 = phi(g2) (means g1 generates G2). A bilinear pairing e is an efficiently computable functions e: G1 X G2 -> G3 such that e(g1, g2) != 1 and e(g1^a, g2^b) = e(g1, g2)^(ab) for all a, b from Z.

This is sometimes called asymmetric pairing. It allows pairings to be constructed on ordinary curves. However, there is no known method to hash to an element of G2 such that its discrete log to some fixed base is unknown. We can still hash to elements of G1 and G3, and perform all other cryptographically useful operations in G2 (picking randome element, multiplications, inversions).

Due to a problem with hashing, the definition is further loosened.

### General bilinear pairing

Let r be a prime. Let G1, G3 be cyclic groups of order r. Let G2 be a group where each element has order dividing r (G2 is not necessarily cyclic). A bilinear pairing e is an efficiently computable function e: G1 X G2 -> G3 such that:

 * e(g1, g2) = 1 for all g2 from G2 iff g1 = 1, and similarly e(g1, g2) = 1 for all g1 from G1 iff g2 = 1 (nondegeneracy)
 * for all g1 from G1 and g2 from G2: e(g1^a, g2^b) = e(g1, g2)^(ab) for all a, b from Z (bilinearity).

Using this definition, the hardness assumptions need to be altered. For different schemes the assumptions (of problem difficulty) might be used in both G1 and G2, or a combination of the two. For example, we may need to assume that given g1, g1^x from G1 and g2 from G2, there is no efficient algorithm to compute g2^x (see [3]).

If r is not prime (composite group order is useful for some cryptosystems [4]), we must be aware that in this case all nondegenerate pairings are not equivalent up to a constant.

### Exponentiation as a bilinear pairing

Given some integer r > 1, let us take G1 = G3 = Z_r\* and G2 = Z_(r-1)\+. Define e: G1 X G2 -> G3 by e(g, a) = g^a.

According to the loosened definition above, this function is bilinear pairing.

## Divisors

Before we see how bilinear pairings are constructed, we first need to have a look at divisors. While rational functions of one variable are defined by zeros and poles, rational functions of two variables are defined by divisors.

Let's have an elliptic curve E: Y^2 = X^3 + A*X + B and a rational function of two variables f(X, Y). Then the numerator of f vanishes on some points of E (zeros) and denominator vanishes on some points of f (poles). Divisor of f is (it is only a notation):

```
div(f) = sum_(all points P of E) n_P * [P] where n_P is multiplicity at point P (poles have negative multiplicity)
```

More generally, we define a divisor on E to be any formal sum:

```
D = sum_(P from E) n_P * [P] with n_P from Z and n_P = 0 for all but finitely many P
```

We further define a degree and sum of a divisor:

```
deg(div(f)) = sum_(all points P of E) n_P
sum(div(f)) = sum_(all points P of E) n_P * P
```

Note that rational functions on E can have different expressions, for example f(x,y) = p(x,y)/q(x,y) = p1(x,y)/q1(x,y). See more about it in [algebraic_geometry.md](https://github.com/miha-stopar/crypto-notes/blob/master/algebraic_geometry.md))

Let's check an example E: y^2 = x^3 - x and the function f(x,y) = x/y on this curve. We see that:

```
x/y = y/(x^2-1)
```

Function f has thus two representations. We say that function is regular at point P if it has representation p/q where q(P) != 0. Thus f is regular at (0,0).


Example: y^2 = x^3 + 1 in GF(13)

```
sage: E = EllipticCurve(GF(13), [0, 1])
sage: E
Elliptic Curve defined by y^2 = x^3 + 1 over Finite Field of size 13
sage: E.points()
[(0 : 1 : 0), (0 : 1 : 1), (0 : 12 : 1), (2 : 3 : 1), (2 : 10 : 1), (4 : 0 : 1), (5 : 3 : 1), (5 : 10 : 1), (6 : 3 : 1), (6 : 10 : 1), (10 : 0 : 1), (12 : 0 : 1)]
```

Let's observe function g(x,y) = x^2 / y. So g(x,y) = p(x,y) / q(x,y) where p(x,y) = x^2 and q(x,y) = y. 

Let's homogenize E and g:

```
E: y^2 * z = x^3 + z^3
g(x,y,z) = x^2 / (y * z)
```

The zeros of p(x,y,z): (0 : 1 : 1), (0 : 12 : 1).
The zeros of q(x,y,z): (4 : 0 : 1), (10 : 0 : 1), (12 : 0 : 1), (0 : 1 : 0).

Note that (0 : 1 : 0) is point at infinity. In these notes point at infinity is denoted simply as 0 (because it is a group unity).

To know the divisor, we need to know the multiplicities (order) of these points. We need some [algebraic_geometry.md](https://github.com/miha-stopar/crypto-notes/blob/master/algebraic_geometry.md) here.

For a domain R that is not a field (because field doesn't have proper ideals) it is the same (see Fulton [6], section about Discrete Valuation Rings):

 * R is Noetherian and local, and the maximal ideal is principal.
 * There is an irreducible element t from R such that every nonzero z from R may be written uniquely in the form z = u * t^n, where u is a unit in R (invertible), t is a generator of maximal ideal, n is called order, t is called uniformizing parameter for R.

What is R in our case? It is O_P(V) - rational functions that are defined in P. The functions that are 0 at P do not have an inverse. V is a variety (in this case elliptic curve). While function field k(V) is obviously a field, O_P(V) is a ring.

It turns out O_P(V) is Noetherian (every ideal finitely generated), local (only one maximal ideal), and the maximal ideal is principal (generated by one element). That means every function z from O_P(V) can be written as z = u * t^n where u does not vanish at P, t vanish at P, and n is called order of the zero.

Let's have a look E: y^2 = x^3 + x at point P = (0, 0). The ideal (rational functions that vanish at P) of O_P(E) is generated by one function. Which? Obviously, functions x and y generate this ideal (all functions that vanish at (0, 0) are something multiplied by x or/and y). On E it holds x = y^2 * 1/(x^2 + 1), thus x is generated by y^2. Thus, y is a generator of the ideal.

So, what is divisor of g(x,y,z) = x^2 / (y * z) on E: y^2 * z = x^3 + z^3 in GF(13)? We saw that the zeros of g are (0 : 1 : 1), (0 : 12 : 1) and poles are (4 : 0 : 1), (10 : 0 : 1), (12 : 0 : 1), (0 : 1 : 0).

Let's check the function x^2 and the order of zero (0 : 1 : 1): it is obvious it is 2 (x is uniformizing parameter, unit u = 1). The same for (0 : 12 : 1).

What about poles (zeros of function y * z)? At (4 : 0 : 1) the uniformizing parameter is y, unit is z, order is 1. The same for (10 : 0 : 1) and (12 : 0 : 1). At (0 : 1 : 0) the uniformizing parameter is z, unit is y, order is 1.

```
div(g) = 2 * (0 : 1 : 0) + 2 * (0 : 12 : 1) - (4 : 0 : 1) - (10 : 0 : 1) - (12 : 0 : 1) - (0 : 1 : 0)
```

Proposition (Washington [9], Proposition 11.1): Let E be an elliptic curve and let f a function on E that is not identically 0. Then:

* f has only finitely many zeros and poles (and the number of zeros and poles is the same)
* deg(div(f)) = 0
* if f has no zeros or poles (so div(f) = 0), then f is a constant.

Proof: Fulton 97

Theorem:

 * Let f and f1 be rational functions on E. If div(f) = div(f1), then there is a nonzero constant c such that f = c * f1.
 * Let D = sum_(P from E) n_P * [P] be a divisor on E. Then D is the divisor of a rational function on E iff: `deg(D) = 0 and sum(D) = 0`.




The divisors of degree 0 form an important subgroup of Div(E), denoted Div^0(E). The function sum gives a surjective homomorphism from divisors of order 0 to the points on elliptic curve:

```
sum: Div^0(E) -> E(K~)
```

It is surjective, because for each P from E(K~), we have an element from Div^0(E) such that:

```
sum([P] - [0]) = P
```

Divisor of a rational function is called a principal divisor.

Lemma (Washington [9], Lemma 11.3): Let P, Q be from E(K~) and a function h on E exists such that `div(h) = [P] - [Q]`. Then P = Q.

Proof (sketch):
Lemma says that functions on E cannot have simple poles (because from the theorem above we know that rational functions on E have the same number of zeros and poles).

Let's say we have function f which does not have a pole or zero at Q. We can construct a function that is a function of h and has the same divisor as f:

```
g(x, y) = ∏_(R on E) (h(x,y) - h(R))^ord_R(f)
```

Note that for all R on E that are not pole or zero of f, the factor in a product is 1. Each factor (where f has zero or pole in R) has a pole in Q and zero in R. We can see that the div(f) = div(g) because the poles in Q cancel each other out (because f has the same number of zeros and poles).

From the theorem above we know that f = g * c for some constant c. That means that f is a function of h, actually it means that every rational function on E that does not have a pole or zero at Q is a function of h (can be expressed with h).

When f has a pole or zero at Q, we observe function f \* h^ord_Q(f). This function has no zero or pole at Q, so we can apply once again the reasoning above and again f is an expression of h. So all functions on E are function of h which leads to a contradiction.

Theorem (Washington [9] Theorem 11.2): Let D be a divisor on E and deg(D) = 0. Then there is a function f on E such that div(f) = D iff sum(D) = 0.

Proof:

We first show that [P1] + [P2] can be replaced by [P1 + P2] + [0] + div(g) for some function g:

There is a line a*x + b*y + c = 0 where P1, P2, P3 lie (P3 is -P1-P2). Let's say P3 = (x3, y3).

```
div(a*x + b*y + c) = [P1] + [P2] + [P3] - 3[0]
div(x-x3) = [P3] + [-P3] - 2[0]
div((a*x + b*y + c)/(x-x3)) = div(a*x + b*y + c) - div(x-x3) = [P1] + [P2] - [P1 + P2] - [0]
[P1] + [P2] = [P1 + P2] + [0] +  div((a*x + b*y + c)/(x-x3))
```

So g = (a*x + b*y + c)/(x-x3). Note that sum(div(g)) = 0.

```
[P1] + [P2] = [P1 + P2] + [0] + div(g)
```

Thus by induction the sum of all the terms in D with positive coefficients equals a single symbol [P] plus a multiple of [0] plus the divisor of a functions. Similarly for the terms with negative coefficients. Therefore:

```
D = [P] - [Q] + n * [0] + div(g1)
```

Since g1 is the quotient (note that div(a) + div(b) = div(a*b)) of products of functions g with sum(div(g)) = 0, we have sum(div(g1)) = 0.

By the proposition above we know deg(div(g1) = 0, thus:

```
0 = deg(d) = 1 - 1 + n + 0 = n
```

So:

```
D = [P] - [Q] + div(g1)
```

Also:

```
sum(D) = P - Q + sum(div(g1)) = P - Q
```

If sum(D) = 0, then P = Q and D = div(g1). Conversely, if D = div(f) for some function f, then [P] - [Q] = div(f/g1). By lemma, we know P = Q.


Corollary (Washington [9] Corollary 11.4): The map

```
sum: Div^0(E) / (principal divisors) -> E(K~)
```

is an isomorpism of groups.

Example (Washington [9] Example 11.4): 
The theorem above gives an algorithm for finding a function with a given divisor (of degree 0 and sum 0). Let's have an elliptic curve y^2 = x^3 + 4*x over F_11 and a divisor D = [(0,0)] + [(2,4)] + [(4,5)] + [(6,3)] - 4[0].

It can quickly be checked that deg(D) = 0 and sum(D) = 0.

```
(0,0) + (2,4) = (2,7)
[(0,0)] + [(2,4)] = [(2,7)] + [0] + div((2*x-y)/(x-2))
```

Thus:

```
D = [(2,7)] + [0] + div((2*x-y)/(x-2)) + [(4,5)] + [(6,3)] - 4*[0]
D = [(2,7)] + div((2*x-y)/(x-2)) + [(4,5)] + [(6,3)] - 3*[0]
```

Further:

```
[(4,5)] + [(6,3)] = [(2,4)] + [0] + div((x+y+2)/(x-2))
```

Thus:

```
D = [(2,7)] + [(2,4)] + div((2*x-y)/(x-2)) + div((x+y+2)/(x-2)) - 2*[0]
```

We can see that:

```
div(x-2) = [(2,7)] + [(2,4)] - 2*[0]
```

Thus:

```
D = div((2*x-y)/(x-2)) + div((x+y+2)/(x-2)) + div(x-2)
D = div((2*x-y) * (x+y+2) / (x-2))
```

## Endomorphisms

See Washington [9], section 2.9.

By an endomorhpism of E, it is meant a homomorphism alpha: E(K~) -> E(K~) that is given by rational functions. So, alpha(P1 + P2) = alpha(P1) + alpha(P2), and there are rational functions R1(x,y), R2(x,y) with coefficients in K~ such that:

```
alpha(x,y) = (R1(x,y), R2(x,y))
```

Obviously, alpha(0) = 0 (point at infinity).

Example: alpha(P) = 2*P. Then alpha(x,y) = (R1(x,y), R2(x,y)) where:

```
R1(x,y) = ((3*x^2 + A)/(2*y))^2 - 2*x
R2(x,y) = ((3*x^2 + A)/(2*y)) * (3*x - ((3*x^2+A)/(2*y))^2) - y
```

We can actually simplify all rational functions on E to the form:

```
R(x,y) = (p1(x) + p2(x) * y) / (p3(x) + p4(x) * y)
```

This is because for all points on E: y^2 = x^3 + a*x + b, so we can replace all even powers of y by a polynomial in x, and replace any odd power of y by y times a polynomial in x.

Moreover, we can multiply by (p3 - p4*y) and obtain:

```
R(x,y) = (q1(x) + q2(x) * y) / q3(x).
```

Since alpha is a homomorphism:

```
alpha(x, -y) = alpha(-(x,y)) = -alpha(x,y)
```

So:

```
R1(x,-y) = R1(x,y)
R2(x,-y) = -R2(x,y)
```

If R1 and R2 are written in the form above (see R(x,y)), then q2(x) = 0 in R1, and q1(x) = 0 in R2:

```
alpha(x,y) = (r1(x), r2(x) * y) where r1(x), r2(x) are rational functions
alpha(x,y) = (p1(x)/q1(x), (p2(x) * y) / q2(x))
```

It can be shown that when r1 is defined in x, then r2 is defined in x, so the rational function is not defined when q1(x) = 0 where r1 = p1/q1.

The degree of alpha is defined as:

```
deg(alpha) = max{deg(p1), deg(q1)}
```

Define alpha != 0 to be a separable endomorphism if the derivative (d/dx)(r1) is not identically zero. This is equivalent to saying that at least one of (d/dx)(p1) and (d/dx)(q1) is not identically zero.

In general, in characteristic p, the map alpha(Q) = p * Q has degree p^2 and is not separable.

An important example of an endomorphism is the Frobenius map:

```
Phi_q(x,y) = (x^q, y^q)
```

Lemma (Washington [9], Lemma 2.20): Let E be defined over F_q. Then Phi_q is an endomorphism of E of degree q and Phi_q is not separable.

Proof: The map is given by rational functions (actually polynomials) and the degree is q. It can be quickly checked that it is a homomorhpism. Since q = 0 in F_q, the derivative of x^q is zero, therefore Phi_q is not separable.

Proposition (Washington [9], Proposition 2.21): Let alpha != 0 be a separable endomorphism of an elliptic curve E. Then deg(alpha) = #Ker(alpha), where Ker(alpha) is the kernel of the homomorhpism alpha: E(K~) -> E(K~). If alpha is not separable, then deg(alpha) > #Ker(alpha).

Proof:

Let's write:

```
alpha(x,y) = (r1(x), y * r2(x)) with r1(x) = p(x)/q(x)
```

Then (d/dx)(r1) != 0 (because it is separable), so (d/dx)(p) * q - p * (d/dx)(q) != 0.

Let S be the set of x from K~ such that (p * (d/dx)(q) - (d/dx)(p) * q) * q = 0.

Let (a,b) be from E(K~) such that:

 * a != 0, b != 0, (a,b) != 0 (point at inifinity)
 * deg(p(x) - a * q(x)) = max{deg(p), deg(q)} = deg(alpha)
 * a is not from r1(S)
 * (a,b) is from alpha(E(K~))

S is a finite set (since both factors of the polynomial are non-zero), so its image under alpha is finite.

Note that the algebraic closure of F_q is ∪_{n=1,...,∞}F_q^n, so it is infinite. If F_q is of characteristic p, F_q~ = F_p~.

The function r1(x) takes infinitely many distinct values as x runs through K~. Since, for each x, there is a point (x,y) from E(K~), we see that alpha(E(K~)) is an infinite set and such (a,b) exists.

We need to show that there are exactly deg(alpha) points (x1,y1) from E(K~) such that alpha(x1,y1) = (a,b). So:

```
p(x1)/q(x1) = a
y1 * r2(x1) = b
```

Since (a,b) != 0 (infinity), we must have q(x1) != 0. As mentioned above where r1 is defined, r2 is defined. Since b != 0 and y1 * r2(x1) = b, we have y1 = b/r2(x1). So x1 determines y1 and we only need to count values of x1.

Thus we need to count the zeros of p(x) - a * q(x). This polynomial is by assumption of degree deg(alpha). We need to show that it does not have multiple roots.

Suppose that (x0,y0) is a multiple root. Then:

```
p(x0) - a * q(x0) = 0
(d/dx)(p)(x0) - a * (d/dx)(q)(x0) = 0
```

So:

```
a * p(x0) * (d/dx)(q)(x0) = a * (d/dx)(p)(x0) * q(x0)
```

So, x0 is from S, which is contrary to the assumption.

Theorem (Washington [9], Theorem 2.22): Let E be an elliptic curve defined over a field K. Let alpha != 0 be an endomorphism of E. Then alpha E(K~) -> E(K~) is surjective.

Proof: Let (a,b) be from E(K~). Since alpha(0) = 0, we may assume that (a,b) != 0. 

Let's see first for the case when p(x) - a * q(x) is not a constant polynomial. The it exists x0 such that p(x0) - a * q(x0) = 0. Since p and q do not have common roots, q(x0) != 0. Now we take for y0 the square root of x^3 + A * x + B. It holds alpha(x0, y0) = (a,b) or alpha(x0, -y0) = (a,b).

Let's now consider the case when p - a*q is constant. Since the kernel of alpha is finite, for a given x only a finite number of points can map to a point with this x. Because E(K~) is infinite and p(x)/q(x) is constant, that means that infinitely many points map into some x, which is a contradiction and means that such a does not exist (that means (a,b) is not in image of alpha). We can quickly show that at most one a exists such that p - a*q is constant. So there exists at most one element which is not the image of alpha. However, let's have (a1,b1) for which alpha(P1) = (a1,b1) and (a1,b1) + (a,b) != (a,+-b). Now there is P2 such that alpha(P2) = (a1,b1) + (a,b) and alpha(P2-P1) = (a,b), alpha(P1-P2) = (a,-b).

Lemma (Washington [9], Lemma 2.24): Let E be the elliptic curve y^2 = x^3 + A * x + B. Fix a point (u,v) on E. Write:

```
(x,y) + (u,v) = (f(x,y), g(x,y))
```

where f(x,y) and g(x,y) are rational functions of x,y and y is regarded as a function of x satisfying dy/dx = (3*x^2 + A) / (2*y). Then:

```
(d/dx)(f(x,y))/g(x,y) = 1/y
```

Proof: Due to lengthy calculation the proof is omitted. Let's just note that:

```
2 * y * dy/dx = 3 * x^2 + A
d/dx f(x,y) = (d/dx)(f) + (d/dy)(f) * (dy/dx)
```

Proposition (Washington [9], Proposition 2.28): Let E be an elliptic curve defined over a field K, and let n be a nonzero integer. Suppose that multiplication by n on E is given by:

```
n(x,y) = (R_n(x), y * S_n(x))
```

for all (x,y) from E(K~), where R_n and S_n are rational functions. Then:

```
(d/dx R_n(x)) / S_n(x) = n
```
 
Therefore, multiplication by n is separable iff n is not a multiple of the characteristic p of the field.


Proposition (Washington [9], Proposition 2.29): Let E be an elliptic curve defined over F_q, where q is a power of the prime p. Let r and s be integers, not both 0. The endomorhpism r * Phi_q + s (where Phi_q is Frobenius endomorphism) is separable iff p does not divide s.

## Rational maps, isogenies and star equations

The notes in this section are taken from [11] which is a self-contained (or as much as it can be) introduction to pairings.

Let E, E1 be two elliptic curves over the same field K. A rational map alpha: E -> E1 is an element of E1(K(E)). Note that K(E) is a function field. An element of E1(K(E)) is (g, f) such that (g(x,y), f(x,y)) satisfy the Weierstrass equation for E1.

Unless alpha is constant, it is surjective. If alpha(0) = 0, then alpha is in fact a group homomorphism, and it is called an isogeny. If E = E1, alpha is called an endomorphism. The endomorphisms that will be used frequently in what follows are multiplication by an integer n, denoted by [n].

A non-constant rational map alpha: E -> E1 induces an injective homomorphism of function fields: alpha\*: K(E1) -> K(E), f1 -> f1 ◦ alpha. The degree of alpha is the degree of the extension [K(E) : alpha\*(K(E1))]. For instance deg([n]) = n^2. Note that the other way around would not work because alpha is not (necessarily) injective - actually that is the reason why there are more elements in K(E) than in alpha\*(K(E1)).

For a point P from E and P1 from E1, there is an integer e_alpha(P), called ramification index, such that ord_P(alpha\*(f1)) = e_alpha(P) * ord_P1(f1) for any f1 from K(E1).

When alpha is an isogeny, e_alpha(P) is independent of P. In this case we have deg(alpha) = e_alpha * $(ker(alpha)). If e_alpha = 1, alpha is called separable ([n] is separable if p does not divide n). If #(ker(alpha)) = 1, alpha is (up to isomorphisms) a power of the purely inseparable Frobenius endomorhpism (x,y) -> (x^q, y^q) of degree and ramification index q.

The ramification index allows to define a homomorhpism alpha\*: Div(E1) -> Div(E) on divisors by:

```
alpha\*([P1]) = Σ_(P from alpha^(-1)(P1)) e_alpha(P)*[P]
```

in such a way that maps alpha\* on functions and divisors are compatible.

Theorem (Upper star equation): If alpha: E -> E1 is a non-constant rational map and f1 from K(E1), then:

```
alpha\*(div(f1)) = div(alpha\*(f1))
```

For a lower star equation we need to refresh what is a field norm. Let's say we have a field K and its finite extension L. Let's choose alpha from L, then m_alpha: L -> L given by m_alpha(x) = alpha * x, is a K-linear transformation. For this linear transformation we can find a corresponding matrix. The norm N_(L/K)(alpha) is defined as the determinant of this linear transformation.

If sigma_1(alpha), ... , sigma_n(alpha) are roots of minimal polynomial of alpha over K, then:

```
N_(L/K) = (Π_(j=1,...,n) sigma_j(alpha))^[L:K(alpha)]
```

where [L:K(alpha)] is a degree of an extension.

Let's observe a map alpha_*: Div(E) -> Div(E1), defined as alpha_*([P]) = [alpha(P)].

A corresponding map on function fields K(E) -> K(E1) can be defined by:

```
alpha_*(f) = (alpha\*)^(-1) (N_(K(E)/alpha\*(K(E1))) (f))
```

The map alpha_* is well-defined because the norm is an element of alpha\*(K(E1)), so a preimage exists. And because alpha\* is injective, so the preimage is unique.

It holds:

```
N_(K(E)/alpha\*(K(E1))) (f) = (Π_(R from ker(alpha) (f ◦ tau_R)^e_alpha
```

where tau_R is translation by R.

Theorem (Lower star equation): If alpha: E -> E1 is a non-constant rational map and f from K(E), then:


```
alpha_*(div(f)) = div(alpha_*(f))
```

### Uniformizer

See above about uniformizing parameter, but briefly: every rational function that is defined at P can be written as z = u * t^n where t is uniformizer (generator of the ideal of all functions that are 0 at P) and u(P) != 0. How to get t?

We have y^2 = x^3 + a*x + b and a point P = (a,b). Functions that are 0 at P are generated by x-a and y-b, but we know that this ideal is principal and has only one generator. Which? Let's denote f(x) = x^3 + a*x + b and expand f(x) as Taylor series in x-a.

```
f(x) = f(a) + (x-a) * c1 + (x-a)^2 * c2 + (x-a)^3 * c3
```

We know f(a) = b^2. So:

```
y^2 - b^2 = (x-a) * c1 + (x-a)^2 * c2 + (x-a)^3 * c3
(y - b) = ((x-a) * c1 + (x-a)^2 * c2 + (x-a)^3 * c3) / (y + b)
(y - b) = (x-a) * (c1 + (x-a) * c2 + (x-a)^2 * c3) / (y + b)
```

So y-b is in the ideal (x-a) and x-a is uniformizer (the generator of the ideal of functions that are 0 at P). But note - if b = 0, this is not true, because y-b = y is in this case written as the product of x-a and some function which is not defined at P.

The uniformizer at points on x-axis is y.

At infinity? 

Function x/y is a uniformizer at 0. We can see this if we move from x,y to x,z coordinates:

```
y^2 = x^3 + a * x + b
y^2 * z = x^3 + a * x * z^2 + b * z^3
z/y = (x/y)^3 + a * x/z * (z/y)^2 + b * (z/y)^3 
(now let's introduce new variables)
z = x^3 + a * x * z^2 + b * z^3 
```

Now we can write:

```
z = x^3 * (1/(1 - a * x * z - b * z^2))
```

We are observing the ring of rational functions that are defined at point at infinity, which is (0,0) in x,z coordinates. The ideal of all rational functions that are 0 at infinity is obviously generated by functions x and z. However, this ideal is principal and have only one generator. From the equation above we can see that z is in the ideal generated by x, so the ideal of rational functions that are 0 at infinity, is generated by function x.

If we transform back to the x,y,z coordinates, the generator x correspond to x/y (see when introducing new variables). So the uniformizer is x/y.

### Leading coefficient

The leading coefficient of a function f at point at infinity 0 is:

```
lc(f) = ((x/y)^(-ord_0(f)) * f)(0)
```

By definition a function f is monic at 0 if lc(f) = 1.

Example: 
Let's have a function (x+y+1)/(x+2) on y^2 = x^3 + 2. We move into x,z coordinates: (x+1+z)/(x+2*z). This function is not defined at infinity: (0+1+0)/(0+2*0). We want to know what is the order of zero of x+2*z at infinity.

The curve in x,z is: z = x^3 + 2*z^3. So:

```
z * (1 - 2*z^2) = x^3
```

Let's rewrite x+2*z:

```
x + 2*z = x + 2 * (x^3/(1-2*z^2) = x * (1 - 2*z^2 + 2*x^2)/(1-2*z^2)
```

It is of the form x * f where f is not 0 at (0,0) and x is uniformizer. So the zero of z+2*z is of order 1.

Now if we want to compute the leading coefficient of (x+y+1)/(x+2), we multiply (x+1+z)/(x+2*z) with x^1 and we get: (x+1+z)*(1-2*z^2)/(1-2*z^2+2*x^2). At (0,0) this function has value 1.

### Canonical form

Canonical form of a function f(x,y): v(x) + y * w(x). Every rational function can be written in canonical form. See Washington [9], section 2.9.

## Weil reciprocity

For D = Σ_(P) n_P * [P] we define support as a set supp(D) = {P; n_P != 0}.

The evaluation of rational function f in divisor:

```
f(Σ_(P) n_P * [P]) = Π_(P) f(P)^n_P
```

To handle common points in the supports, the tame symbol of two functions f and g is defined as:

```
<f,g>_P = (-1)^(ord_P(f)*ord_P(g)) * (f^ord_P(g) / g^ord_P(f)) (P)
```

Theorem (Generalised Weil reciprocity): If f, g from K(E), then:

```
Π_(P from E(K~)) <f,g>_P = 1
```

In particular, if supp(f) ∩ supp(g) = {}, then:

```
f(div g) = g(div f)
```

## Weil pairing

The set of points of order m (torsion points):

```
E[m] = {P from E: m * P = 0}
```

It holds (Silverman [8] Corollary III.6.4): Let E be an elliptic curve over F_p and assume that p does not divide m. Then there exists a value of k such that:

```
E(F_p^(j*k))[m] = Z_m x Z_m for all j >= 1
```

If m is prime and if K is a field, then we may view E[m] as a 2-dimensionl vector space over Z_m. Even if m is not prime, there is still {P1, P2} such that each P from E[m] can be written P = a * P1 + b * P2 where a, b are unique from Z_m.


Weil pairing takes as input a pair of points P, Q from E[m] and gives as output an m-th root of unity. Let's denote mi_m = {x from K~; x^m = 1}.

```
e_m : E[m] x E[m] -> mi_m
```

Weil pairing is a major tool in the study of elliptic curves - it can be for example used to prove Hasse's theorem on the number of points on an elliptic curve over a finite field.

There are multiple equivalent definitions of Weil pairing.

### First definition of Weil pairing

Definition (see for example section 5.8.3 in [5]): Let be P, Q from E[m]. Let f_P and f_Q be rational functions on E satisfying (we know due to the theorem above that such functions exist):

```
div(f_P) = m * [P] - m * [0]
div(f_Q) = m * [Q] - m * [0]
```

The Weil pairing of P and Q is the quantity:

```
e_m(P, Q) = ( f_P(Q + S) / f_P(S) ) / ( f_Q(P - S) / f_Q(-S) )
```

where S from E is any point satisfying S not from {0, P, -Q, P-Q}.

Now, is Weil pairing well-defined - that means independent from the choice of f_P, f_Q and auxiliary point S? 

We can see that it is independent of the choice of f_P and f_Q because the rational functions on elliptic curve (see Theorem above) are by divisor defined up to a constant and in the Weil pairing the constants cancel each other out.

We can see that it is independent of the choice of S if have a look at the pairing as the function of S and compute its divisor:

```
F(S) = ( f_P(Q + S) / f_P(S) ) / ( f_Q(P - S) / f_Q(-S) )
```

It turns out the divisor is 0, thus F(S) is constant.

### Second definition of Weil pairing

Let's choose T from E[n]. There exists function f such that:

```
div(f) = n[T] - n[0]
```

Let's choose T1 from E[n^2] such that n*T1 = T (that means n^2 * T1 = 0). There exists function g such that (because degree and sum are 0):

```
div(g) = sum_(R from E[n]) ([T1 + R] - [R])
```

Let's observe the function fn: P -> f(n*P). We can see that zeros of this function are P for which: n*P = T. These are T1 + R where R from E[n]. Thus:

```
div(fn) = n * sum_(R from E[n])([T1 + R]) - n * sum_(R from E[n])([R]) = div(g^n)
```

By multiplying f by a suitable constant, we may assume that:

```
fn = g^n
```

Let S be from E[n] and P from E[K~], then:

```
g(P+S)^n = f(n*(P+S)) = f(n*P) = g(P)^n
```

Therefore, g(P+S)/g(P) is from mi_n. It can be shown that g(P+S)/g(P) is independent of P.

Weil pairing:

```
e_n(S, T) = g(P + S)/g(P)
```

Example of using upper star equation:

```
div(f ◦ [n]) = div([n]*(f)) = [n]*(div(f)) = [n]*(n[P]-n[0]) = n * Σ_(R from E[n]) ([T + R] - [R]) 
```

### Weil pairing properties

#### Bilinearity

e_n is bilinear in each variable:

```
e_n(S1 + S2, T) = e_n(S1, T) * e_n(S2, T)
e_n(S, T1 + T2) = e_n(S, T1) * e_n(S, T2)
```

for all S, S1, S2, T, T1, T2 from E[n].

To prove linearity in the first variable (use P+S1 as P in the second factor):

```
e_n(S1, T) * e_n(S2, T) = (g(P + S1) / g(P)) * (g(P + S1 + S2) / g(P+S1)) = g(P + S1 + S2) / g(P)
```

For linearity in the second variable:

```
e_n(S, T1 + T2) = g3(P+S)/g3(P)
e_n(S, T1) = g1(P+S)/g1(P)
e_n(S, T2) = g2(P+S)/g2(P)
T3 := T1 + T2
```

Let be f1, f2, f3 functions that corresponed to g1, g2, g3. That means f1(T1) = 0, f2(T2) = 0, f3(T3) = 0 and div(f1) = n[T1] - n[0] ...

There exists function h such that:

```
div(h) = [T3] - [T1] - [T2] + [0]
```

It holds:

```
div(f3/(f1*f2)) = div(h^n)
```

So:

```
f3 = c * f1 * f2 * h^n where c is constant
f3(n*P) = c * f1(n*P) * f2(n*P) * h(n*P)^n
g3(P)^n = c * g1(P)^n * g2(P)^n * h(n*P)^n
g3 = c * g1 * g2 * (h o n)
```

So:

```
e_n(S, T1 + T2) = (g3(P+S)/g3(P) = g1(P+S) * g2(P+S) * h(n * (P+S))) / (g3(P) * g1(P) * g2(P) * h(n*P)) = e_n(S, T1) * e_n(S, T2)
```

because n*S = 0.

#### e_n(T, T) = 1 for all T from E[n]

Let's define f_j as function: P -> f(P + j*T) and observe function ∏_(j=0,...,n-1) (f_j):

```
div(∏_(j=0,...,n-1) (f_j)) = Σ_(j=0,...,n-1) (n*[(1-j)*T] - n*[-j*T]) = 0 
```

because n*T = 0 and thus sum is 0 (degree is obviously 0). Thus this function is constant.

Now let's define g_j: P -> g(P + j*T1) where n*T1 = T. It holds:

```
∏_(j=0,...,n-1) (g_j^n)) = ∏_(j=0,...,n-1) (f_j ◦ [n])) where [n] is a function P -> n*P
```

As the product of f_j is constant, the function on the right side is constant and thus the product of g_j^n is constant. So ∏_(j=0,...,n-1) (g_j) is constant as well and:

```
∏_(j=0,...,n-1) g(P + T1 + j*T1) = ∏_(j=0,...,n-1) g(P + j*T1)
```

After canceling the common terms:

```
g(P + n*T1) = g(P)
g(P + T) = g(P)
```

#### Alternation

```
e_n(T, S) = e_n(S, T)^(-1) for all S, T from E[n]
```

Proof:

```
1 = e_n(T+S, T+S) = e_n(T, T) * e_n(T, S) * e_n(S, T) * e_n(S, S) = e_n(T, S) * e_n(S, T)
```

#### Nondegeneracy

e_n is nondegenerate in each variable. This means that if e_n(S, T) = 1 for all T from E[n] then S = 0 and also that if e_n(S, T) = 1 for all S from E[n] then T = 0.

Due to alternation it is enough to prove non-degeneracy with respect to the first argument.

Let's say e_n(S, T) = 1 for all T. Then g(P+S) = g(P). Let's observe now the equation from above (lower star ... ):

```
N_(K(E)/alpha\*(K(E1))) (f) = (Π_(R from ker(alpha) (f ◦ tau_R)^e_alpha
```

If alpha is [n], we have ramification index e_alpha = 1 and #ker(alpha) = n. Due to g(P+S) = g(P), g ◦ tau_S = g. Thus:

```
N_(K(E)/alpha\*(K(E1))) (g) = g^n
```

So we have a function h such that g = h ◦ [n]. Also by definition of Weil pairing: g^n = f ◦ [n]. So

```
f ◦ [n] = (h ◦ [n])^n
div(f ◦ [n]) = n * div(h ◦ [n])
```

By upper star equation:

```
div(f) = m * div(h) 
div(h) = [T] - [0]
```

We know that such h does not exist, so T = 0.

#### e_n(h(S), h(T)) = h(e_n(S, T)) 

e_n(h(S), h(T)) = h(e_n(S, T)) for all automorphisms h of K~ such that h is the identity map on the coefficients of E (if E is in Weierstrass form, this means h(A) = A, h(B) = B).

Proof: 

Let's denote with f^h the function obtained by applying h on the coefficients on f. We do the same for g (f and g from Weil pairing definition).

Let's say f corresponds to T, then f^h corresponds to h(T), because div(f^h) = n*[h(T)] - n*[0] and h is homomorphism. Similarly for g and g^h.

```
h(e_n(S, T)) = h(g(P+S)/g(P))
e_n(h(S), h(T)) = g^h(h(P) + h(S)) / g^h(h(P)) = g^h(h(P+S)) / g^h(h(P)
```

Both are the same due to the definition of g^h and h being homomorphism.

#### Compatibility with isogenies

```
e_n(alpha(S), alpha(T)) = e_n(S, T)^deg(alpha)
```

for all separable endomorphisms alpha of E.

Proof:

Let {Q_1,...,Q_k} = Ker(alpha). Since alpha is separable, k = deg(alpha). Let:

```
div(f_T) = n[T] - n[0]
div(f_alpha(T)) = n[alpha(T)] - n[0]
g_T^n = f_T ◦ n
g_alpha(T)^n = f_alpha(T) ◦ n
```

Let tau_Q denote adding Q. Then:

```
div(f_T ◦ tau_(-Q_i)) = n[T+Q_i] - n[Q_i] 
```

Therefore:

```
div(f_alpha(T) ◦ alpha) = n * Σ_(alpha(T1) = alpha(T)) [T1] - n * Σ_(alpha(Q) = 0) [Q] = n * Σ ([T + Q_i] - [Q_i]) = div(Π f_T ◦ tau_(-Q_i))
```

For each i, choose Q1_i with n*Q1_i = Q_i. Then:

```
g_T(P - Q1_i)^n = f_T(n*P - Q_i) 
```

And:

```
div(Π (g_T ◦ tau_(-Q1_i))^n) = div(Π  f_T ◦ tau_(-Qi) ◦ n) = div(f_alpha(T) ◦ alpah ◦ n) = div(f_alpha(T) ◦ n ◦ alpha) = div(g_alpha(T) ◦ alpha)^n
```

Thus:

```
e_n(alpha(S), alpha(T)) = g_alpha(T)(alpha(P+S)) / g_alpha(T)(alpha(P)) = Π (g_T(P+S-Q1_i) / g_T(P - Q1__i)) = Π e_n(S, T) = e_n(S, T)^deg(alpha)
```

For Frobenius endomorphism:

```
e_n(Phi_q(S), Phi_q(T)) = e_n(S, T)^q
```

### Third definition of Weil pairing 

For P, Q from E[n]-{0}, P != Q, let f_P and f_Q be such that:

```
f_P = n[P] - n[0]
f_Q = n[Q] - n[0]
```

Then:

```
e_n(P,Q) = (-1)^n * f_P(Q)/f_Q(P) * (f_Q/f_P)(0)
```

If f_P and f_Q are chosen monic at 0, then:

```
e_n(P,Q) = (-1)^n * f_P(Q)/f_Q(P)
```

For P = Q or one or both of P and Q being 0, the definition needs to be completed by e_n(P,Q) = 1.

### Equivalence of the second and third definition

Let P_0 and Q_0 be such that:

```
n*P_0 = P
n*Q_0 = Q
```

Let g_P be the function, monic at 0, such that:

```
div(g_P) = Σ_(R from E[n]) ([P_0 + R] - [R])
```

Similarly for g_Q.

Let h_Q be the function, monic at 0, such that:

```
div(h_Q) = (n-1)[Q_0] + [Q_0 - Q) - n[0]
```

Let: 

```
H_Q = Π_(R from E[n])(h_Q ◦ tau_R)
```

By generalised Weil reciprocity:

```
Π_(S from supp(div(g_P)) ∪ supp(div(h_Q))) <g_P, h_Q>_S = 1
```

If P != Q, then supp(div(g_P)) ∩ supp(div(h_Q)) = {0}, and we can easily compute:

```
<g_P, h_Q>_Q_0 = g_P^(n-1)(Q_0)
<g_P, h_Q>_(Q_0-Q) = g_P(Q_0-Q)
<g_P, h_Q>_(P_0+R) = h_Q^(-1)(P_0+R) for R from E[n]
<g_P, h_Q>_(R) = h_Q(R) for R from E[n] - {0}
<g_P, h_Q>_0 = (-1)^n * (h_Q/g_P^n)(0) = (-1)^n since g_P and h_Q are monic at 0
```

Multiplying:

```
1 = (-1)^n * g_P^n(Q_0)/g_Q^n(P_0) * e_n(P, Q)^-1
```

Note that:

```
g_P^n = c^(-1) * [n]*(f_P)
c = lc([n]*(f_P)) = ((f_P ◦ [n])* X^n/Y^n)(0)
```

### Fourth definition of Weil pairing

Let S, T from E[n]. Let D_S and D_T be divisors of degree 0 such that:

```
sum(D_S) = S
sum(D_T) = T
```

and such that D_S and D_T have no points in common. Let f_S and f_T be functions such that:

```
div(f_S) = n * D_S
div(f_T) = n * D_T
```

Then Weil pairing is given by:

```
e_n(S, T) = f_T(D_S) / f_S(D_T)
```

Recall that f(Σ a_i * P) = Π f(P)^a_i.


Example (Washington [9] Example 11.5): Let's have y^2 = x^3 + 2 over GF(7). Let's compute e_3((0,3),(5,1)).











[1] A. Menezes, “An Introduction to Pairing-Based Cryptography,” in 1991 Mathematics Subject Classification, Primary 94A60, 1991.

[2] Lynn, Ben. On the implementation of pairing-based cryptosystems. Diss. Stanford University, 2007.

[3] D. Boneh, B. Lynn, and H. Shacham. Short signatures from the Weil pairing. In Asiacrypt, volume 2248 of Lecture Notes in Computer Science, pages 514+, 2001.

[4] D. Boneh, E.-J. Goh, and K. Nissim. Evaluating 2-DNF formulas on ciphertexts. In Theory of Cryptography ’05, volume 3378 of Lecture Notes in Computer Science, pages 325–341. Springer-Verlag, 2005.

[5] Hoffstein, Jeffrey, et al. An introduction to mathematical cryptography. Vol. 1. New York: springer, 2008.

[6] Fulton, William. "Algebraic curves." An Introduction to Algebraic Geom (2008): 54.

[7] Shafarevich, Igor Rostislavovich, and Kurt Augustus Hirsch. Basic algebraic geometry. Vol. 2. Berlin: Springer-Verlag, 1994.

[8] Silverman, Joseph H. The arithmetic of elliptic curves. Vol. 106. Springer Science & Business Media, 2009.

[9] Washington, Lawrence C. Elliptic curves: number theory and cryptography. CRC press, 2008.

[10] Fulton, William. "Algebraic curves." An Introduction to Algebraic Geom (2008): 54.

[11] Enge, Andreas. "Bilinear pairings on elliptic curves." arXiv preprint arXiv:1301.5520 (2013).

