[]()#modular-arithmetic #cryptography 

---

A system of arithmetic for integers, where numbers "wrap around" when reaching a certain value, called the modulus. Another way of saying this, is that when we divide the integer `a` by `m`, the remainder is `b`. This tells you that if m divides a (this can be written as `m | a`) then `a ≡ 0 mod m`.

### Finite Fields
Reference: https://www.nku.edu/~christensen/Introduction%20to%20finite%20fields.pdf

### Fermat's Little Theroem
Lets say we pick `p = 17`. Calculate `3¹⁷ mod 17`. Answer is 3.
Now do the same but with `5¹⁷ mod 17`.  Answer is 5
What would you expect to get for `7¹⁶ mod 17`? Answer is 1
  
This interesting fact is known as Fermat's little theorem.

Formally, Fermat's Little Theorem is a fundamental result in number theory that states that if p is a prime number and a is any integer not divisible by p, then:

`a^(p-1) ≡ 1 (mod p)`

### Multiplicative Inverse
For all elements `g` in the finite field `Fp`, there exists a unique integer `d`. 
Such that `g * d ≡ 1 mod p`. This is the multiplicative inverse of `g`

How to find `d` for any `g`?

Reference: https://cryptohack.org/challenges/mdiv/solutions/#Hon

Looking again at Fermat's little theorem
If p is prime, for every integer a: `pow(a, p) = a mod p`
and, if p is prime and a is an integer coprime with p: `pow(a, p-1) = 1 mod p`
We can do some magic like this:

`a^(p-1) = 1 (mod p)`
`a^(p-1) * a^-1 = a^-1 (mod p)`
`a^(p-2) * a * a^-1 = a^-1 (mod p)`
`a^(p-2) * 1 = a^-1 (mod p)`

So finally we have:

`a^(p-2) = a^-1 (mod p)`

So, doing a^(p-2) and then (mod p) we can achieve `d` which is our multiplicative inverse.

```python
def calc_d(a,p):
	d = pow(a,p-2) % p
	return d
```


### Quadratic Residue

An integer `x` is a _Quadratic Residue_ if there exists an `a` such that `a² = x mod p`. If there is no such solution, then the integer is a _Quadratic Non-Residue_. We say, `a` is the square root of `x`. 

Let modulo `p = 29`. We can take the integer `a = 11` and calculate `a² = 5 mod 29`.  
As `a = 11, a² = 5`, we say the square root of `5` is `11`. 
	
*Naive way of checking is_quadratic_residue*
```python
def is_quadratic_residue(x: int, p: int) -> tuple:
	"""
	Check if x, is a quadratic residue, if thre exists an integer a, in the finite field, Fp. 
	Such that a^2 = x mod p. Here, a is the square root of x.
	Returns (True, a) if x is a quadratic residue, else returns (False, None)
	"""
	
	for a in range(1, p):
		if (a**2) % p == x:
			return True, a
	else:
		return False, None
```

*With Legendre Symbol* 
```python
def is_quadratic_residue(x: int, p: int) -> bool:
	"""
	Check if x, is a quadratic residue, if there exists an integer a, in the finite field, Fp. 
	Such that a^2 = x mod p. We can say, a is the square root of x.

	Uses Legendre's Symbol for checking. (x / p) ≡ pow(x, (p-1)//2, p) obeys:
		(x / p) = 1 if x is a quadratic residue and x ≢ 0 mod p
		(x / p) = -1 if x is a quadratic non-residue mod p
		(x / p) = 0 if x ≡ 0 mod p

	Returns True if x is a quadratic residue, else False.
	"""
	
	l = pow(x,(p-1)//2, p)

	if l == 1:
		return True
	else:
		return False
```


### Square Root of Quadratic Residue (Tonelli-Shanks)

The square root, `x`, of a quadratic residue, `a`, in a finite field `Fp` is such that, `x^2 = a (mod p)`  The best algorithm to calculate the square roots of a is the Tonelli-Shanks algorithm. (The main use-case for this algorithm is finding elliptic curve co-ordinates).

All primes that aren't 2 are of the form `p ≡ 1 mod 4` or `p ≡ 3 mod 4`, since all odd numbers obey these congruences. 

**Case 1: `p ≡ 3 mod 4`** 

In the `p ≡ 3 mod 4` case, a really simple formula for computing square roots can be [derived](https://crypto.stackexchange.com/a/20994) directly from Fermat's little theorem. In python it looks like the following:
```python
if p % 4 == 3:
        x = pow(a, (p+1)//4, p)
        return [x, p-x]
```

**Case 2: `p ≡ 1 mod 4`** 

In this case, a more general algorithm is required. Here's where the Tonelli-Shanks algorithm comes in, which calculates `r` in a congruence of the form `r^2 ≡ a mod p`. The following python code implements the Tonelli-Shanks algorithm. Referenced from [here](https://codereview.stackexchange.com/questions/43210/tonelli-shanks-algorithm-implementation-of-prime-modular-square-root).

```python
def sqrt_quadratic_residue(a: int, p: int) -> list:
    """
    Finds square roots, of quadratic resude, x, in the finite field, Fp
    Such that a^2 = x (mod p). Where a is the square root.
    Uses the Tonelli-Shanks algorithm. 
    https://en.wikipedia.org/wiki/Tonelli%E2%80%93Shanks_algorithm

    Reference: https://codereview.stackexchange.com/questions/43210/tonelli-shanks-algorithm-implementation-of-prime-modular-square-root

    Returns list of square roots.
    """

    a %= p

    # Simple case
    if a == 0:
        return [0]
    if p == 2:
        return [a]
    
    # Check if a is a quadratic residue
    if not quadratic_residue(a, p):
        return []

    # Simple case of p = 3 (mod 4)
    if p % 4 == 3:
        x = pow(a, (p+1)//4, p)
        return [x, p-x]

    # Factor p-1 on the form q * 2^s (with Q being odd)
    q, s = p - 1, 0
    while q % 2 == 0:
        s += 1
        q //= 2

    # Select a z which is a quadratic non-residue mod p
    z = 1
    while quadratic_residue(z, p):
        z += 1
    c = pow(z, q, p)

    # Search for a solution
    x = pow(a, (q+1)//2, p)
    t = pow(a, q, p)
    m = s
    while t != 1:
        # Find the lowest i such that t^(2^i) = 1
        i, e = 0, 2
        for i in range(1, m):
            if pow(t, e, p) == 1:
                break
            e *= 2

        # Update next value of b, c, t and m
        b = pow(c, 2**(m-i-1), p)
        x = (x * b) % p
        t = (t * b * b) % p
        c = (b * b) % p
        m = i
    
    return [x, p-x]
```


### Chinese Remainder Theorem

The Chinese Remainder Theorem gives a unique solution to a set of linear congruences if their moduli are coprime.  This means, that given a set of arbitrary integers `ai`, and pairwise coprime integers `ni`, such that the following linear congruences hold:

```
x ≡ a1 mod n1  
x ≡ a2 mod n2  
...  
x ≡ an mod nn
```

In other words, it tells us how to find a number that satisfies a set of equations of the above form.
There is a unique solution `x ≡ a mod N` where `N = n1 * n2 * ... * nn`.

To find this solution, we can use the following steps:

1. Compute the product N = n₁*n₂*...*nₖ of all the n values.
2. For each i from 1 to k, compute the value Mᵢ = N/nᵢ, which is the product of all the n values except for nᵢ.
3. For each i from 1 to k, compute the inverse yᵢ of Mᵢ modulo nᵢ. In other words, find an integer yᵢ such that Mᵢ*yᵢ ≡ 1 (mod nᵢ).
4. The solution x is then given by the formula: `x ≡ a₁*M₁*y₁ + a₂*M₂*y₂ +...+ aₖ*Mₖ*yₖ (mod N)`

Implementation in python3:

```python
def chinese_remainder_theorem(a, n):
    """
    Solves a system of linear congruences using the Chinese Remainder Theorem.
    Inputs:
        - a: list of integers representing the residues in the congruences
        - n: list of integers representing the moduli in the congruences
          (assumed to be pairwise coprime)
    Returns:
        - An integer x that satisfies all of the congruences.
    """
    N = 1
    for ni in n:
        N *= ni
    x = 0
    for ai, ni in zip(a, n):
        Mi = N // ni
        yi = pow(Mi, -1, ni)
        x += ai * Mi * yi
    return x % N
```

