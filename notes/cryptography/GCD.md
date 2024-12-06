#gcd #cryptography 

---

The Greatest Common Divisor (GCD), sometimes known as the highest common factor, is the largest number which divides two positive integers (a,b).  
  
For `a = 12, b = 8` we can calculate the divisors of a: `{1,2,3,4,6,12}` and the divisors of b: `{1,2,4,8}`. Comparing these two, we see that `gcd(a,b) = 4`.  
  
Now imagine we take `a = 11, b = 17`. Both `a` and `b` are prime numbers. As a prime number has only itself and `1` as divisors, `gcd(a,b) = 1`.  
  
We say that for any two integers `a,b`, if `gcd(a,b) = 1` then `a` and `b` are coprime integers.  
  
If `a` and `b` are prime, they are also coprime. If `a` is prime and `b < a` then `a` and `b` are coprime.

**For understanding extended egcd aglo:**
https://en.wikipedia.org/wiki/Euclidean_algorithm
http://www-math.ucdenver.edu/~wcherowi/courses/m5410/exeucalg.html

Extended Euclidian algorithm in python to return gcd(a,b), u, and v such that `au + bv = gcd(a,b)``

```python
def gcd(a: int, b: int) -> int:
    """
    Extended Euclidian algorithm to calculate the GCD of 2 numbers
    """
    
    while True:
        if max(a,b) % min(a,b) == 0:
            return min(a,b)
        a, b = min(a,b), max(a,b) % min(a,b)
```

**Reference:** https://anh.cs.luc.edu/331/notes/xgcd.pdf