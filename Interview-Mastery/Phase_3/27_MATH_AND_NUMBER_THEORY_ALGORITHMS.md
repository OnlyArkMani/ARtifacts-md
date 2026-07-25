# 27_MATH_AND_NUMBER_THEORY_ALGORITHMS

1. Introduction

What this concept is

Math and number-theory algorithms encompass gcd/lcm computations, modular arithmetic, fast exponentiation, modular inverse, primality testing (Miller–Rabin), sieve of Eratosthenes, factorization (Pollard's Rho), multiplicative functions, and combinatorics (nCr modulo prime, Lucas theorem).

Why it exists

Many algorithmic problems rely on arithmetic under modulo constraints, counting combinations under mod, primality/factorization, and discrete logarithm variants. Efficient math primitives enable solving large-input tasks in contests and real systems.

What problem it solves

- Fast modular exponentiation and inverses for cryptography and counting under mod
- Primality tests and factorization for number-theory tasks
- Efficient combinatorics under modulo constraints

Real-world analogy

Modular arithmetic akin to clock arithmetic; multiplication wraps around modulus.

Where it is used in industry

- Cryptography (RSA, ECC), hashing, randomized algorithms, computational number theory, finance (modular models)


2. Intuition Section

Modular exponentiation

- Use exponent binary decomposition to compute x^y mod m in O(log y) multiplications via repeated squaring

GCD/extended GCD

- Euclid’s algorithm reduces gcd(a,b) via gcd(b, a mod b); extended version finds coefficients x,y with ax+by=gcd(a,b)

Primality testing

- Miller–Rabin probabilistic test detects composites quickly; deterministic base sets for 64-bit range exist


3. Core Theory

Modular inverse

- For prime mod p, inverse of a exists if gcd(a,p)=1 and inverse = a^{p-2} mod p by Fermat’s little theorem
- Extended GCD computes inverse in general modulus

Combination modulo prime

- Precompute factorials and inverse factorials up to n to answer nCr mod p in O(1)

Pollard's Rho

- Randomized factorization effective for large semiprimes in practice; uses polynomial sequences and cycle detection


4. Java Implementation (mod pow & extgcd)

```java
long modPow(long a, long e, long mod){ long res=1%mod; a%=mod; while(e>0){ if((e&1)==1) res = (res*a)%mod; a = (a*a)%mod; e>>=1; } return res; }

long extgcd(long a, long b, long[] out){ if(b==0){ out[0]=1; out[1]=0; return a; } long g = extgcd(b, a%b, out); long x = out[1]; long y = out[0] - (a/b)*out[1]; out[0]=x; out[1]=y; return g; }

long modInv(long a, long mod){ long[] out=new long[2]; long g=extgcd(a,mod,out); if(g!=1) return -1; long inv = (out[0]%mod+mod)%mod; return inv; }
```

Explain: modular inverse via extended GCD works for non-prime mods too when inverse exists.


5. Python Implementation (sieve and Miller–Rabin sketch)

Sieve of Eratosthenes

```python
def sieve(n):
    isprime = [True]*(n+1); isprime[0]=isprime[1]=False
    for i in range(2,int(n**0.5)+1):
        if isprime[i]:
            step = i
            for j in range(i*i, n+1, step): isprime[j]=False
    return [i for i,pr in enumerate(isprime) if pr]
```

Miller–Rabin (deterministic bases for 64-bit)

Outline: decompose n-1 = d * 2^s, test random bases a with a^d mod n and witness loops; use known bases for deterministic checks up to 2^64.


6. Internal Working

Why fast exponentiation works

Binary decomposition reduces exponent multiplications to O(log e); modular reductions keep numbers bounded

Why Euclid's algorithm is fast

Each iteration reduces argument size significantly; runs in O(log min(a,b)) steps


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| gcd | O(log min(a,b)) |
| modPow | O(log e) multiplications |
| sieve | O(n log log n) |
| Miller–Rabin test | O(k log^3 n) per base roughly depending on modular exponentiation |


8. Space Complexity

- Sieve uses O(n) memory
- Factorials precomputation O(n)


9. Common Interview Questions

- Implement modular exponentiation and modular inverse
- Count combinations modulo prime using factorial precomputation
- Use sieve to list primes up to n


10. Common Mistakes

Mistake: integer overflow when multiplying before modulo in languages without big integers
Fix: use 128-bit where available or modular multiplication safe routines

Mistake: incorrect base selection in Miller–Rabin causing false positives in adversarial tests


11. Real Interview Traps

Trap: applying Fermat inverse when modulus isn't prime

Trap: implementing naive factorization and getting TLE on large semiprimes


12. Real World Applications

- Cryptography, primality testing for key generation, combinatorics in probability and hashing


13. Common LeetCode Problems

- Super Ugly Number variants, modular counting problems, combinatoric modulus tasks


14. Pattern Recognition

- Use sieve for offline prime generation up to moderate n
- Use Miller–Rabin for primality testing in large ranges
- Use Pollard-Rho for practical factorization when needed


15. 5 Minute Revision

- Modular arithmetic primitives: modPow, modInv (Fermat/extGCD), sieve, Miller–Rabin
- Precompute factorials + inverses for fast nCr mod p


---
