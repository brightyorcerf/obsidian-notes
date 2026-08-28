
int lb_idx = lower_bound(arr.begin(), arr.end(), n) - arr.begin();
int ub_idx = upper_bound(arr.begin(), arr.end(), n) - arr.begin();

`to_string(n)` = convert `int` to `string`
`stoi(s)` = convert `string` to `int`

memset fills a contiguous block of memory with a specific byte value
`void* memset(void* ptr, int value, size_t num);`

• Substrings / Subarrays (Contiguous): `O(n²)`
• Subsequences (Non-contiguous, Order kept): `O(2ⁿ)`

- **`vector<int> arr(5):`** Creates 1 vector holding 5 plain integers (1D).
- **`vector<int> arr[5]:`** Creates a fixed-size C-array of 5 vectors stored on the stack (2D).
- **`vector<vector<int>> arr(5):`** Creates 1 dynamic vector holding 5 vectors stored on the heap (2D).

---

### Modulo 

finds remainder when one integer is divided by another
$$a \equiv r \pmod m \implies a = q \cdot m + r \quad (0 \le r < m)$$
(when you divide a number by m the divisor is r, where r is strictly lesser than m)
example) $14 \equiv 3 \pmod 2$

#### Common Algebraic Operations

These identities allow you to perform arithmetic step-by-step to prevent 64-bit integer overflow in Codeforces problems:

- Addition: $(a + b) \pmod m = ((a \pmod m) + (b \pmod m)) \pmod m$
- Subtraction: $(a - b) \pmod m = ((a \pmod m) - (b \pmod m) + m) \pmod m$ 
(adds $m$ to avoid negative remainders)
- Multiplication: $(a \cdot b) \pmod m = ((a \pmod m) \cdot (b \pmod m)) \pmod m$
- Exponentiation: $a^b \pmod m$ computed in $O(\log b)$ time using Binary Exponentiation

#### Key Properties & Tricks

1. Divisibility & Digital Root ($\pmod 9$ and $\pmod 3$)
Property: Any integer $n$ in base 10 satisfies $n \equiv \text{sum of digits of } n \pmod 9$.
Digital Root Formula:

> A digital root is the single-digit value obtained by repeatedly adding the digits of a given number until only one digit remains
$$\text{dr}(n) = \begin{cases} 0 & \text{if } n = 0 \\ 9 & \text{if } n \ne 0 \text{ and } n \pmod 9 = 0 \\ n \pmod 9 & \text{otherwise} \end{cases}$$

Shortcut: $\text{dr}(n) = 1 + ((n - 1) \pmod 9)$ for $n > 0$.
CP Application: Instant $O(1)$ checking for divisibility, digital root games, and parity/invariant checks on large inputs given as strings.
 
2)  Negative Modulo: -7 % 3 evaluates to -1 (truncation toward zero)

#### Modular Inverse

In regular math, division is just multiplying by a fraction (reciprocal) =>
dividing by 3 == multiplying by 1/3. Modular inverse is the modular arithmetic equivalent of dividing. On a standard 12h clock, if it is 9 o'clock and you add 4h, it isn't 13 o'clock, it's 1 o'clock. You wrapped around.

- Math on a Loop: Modular arithmetic operates on a fixed loop ($0$ to $M-1$) instead of an infinite straight line. Standard division breaks because fractions/floats lose precision and break modulo congruences.
- Replacing Division: Division by $B$ is impossible under modulo. Instead, we replace A/B with $(A \times B^{-1}) \pmod M$.
- Definition: The modular inverse $B^{-1}$ is a whole number $X$ such that:

$$(B \times X) \equiv 1 \pmod M$$

- Key Condition: $B^{-1}$ exists if and only if $\gcd(B, M) = 1$ ($B$ and $M$ are coprime).

##### Extended Euclidean Approach 

Euclidean Algorithm is a fast way to find the GCD of two numbers
$\text{gcd}(a, b) = \text{gcd}(b, a \pmod b)$ where $\text{gcd}(a, 0) = a$

```
int gcd(int a, int b){
	if b == 0 ? a : gcd(b, a%b);
}
```

Modulo inverse => $a \cdot x \equiv 1 \pmod m$
=> $a \cdot x - m \cdot y = 1$ (linear diophantine equation)
By Bézout's Identity, an equation of the form $ax + my = d$ only has integer solutions for $x$ and $y$ if $d$ is a multiple of $\gcd(a, m)$. In other words  $ax + my = gcd(a, m)k$
Since d =1 here therefore $\gcd(a, m) \text{ MUST equal } 1$ 

```
// solves a*x + b*y = gcd(a, b)
// returns gcd(a, b) and updates x, y by reference

ll extendedEuclidean(ll a, ll b, ll& x, ll& y){ 
	// x and y called by reference
	
	if(b == 0){ // base case
		x = 1; 
		y = 0;
		return a;
	}
	ll x1, y1;
	ll g = extendedEuclidean(b, a%b, x1, y1);
	
	x = y1; 
	y = x1 - y1 * (a/b);
	
	return g;
}

int main(){
	ll a, m, x, y; cin >> a >> m;
	ll g = extendedEuclidean(a, m, x, y);
	if(g != 1) cout << "no solution";
	else{
		x = (x % m + m) % m;
		cout << x;
	}
}
```

### Fermat's little theorem

[visualized](https://www.youtube.com/watch?v=XPMzosLWGHo)
$$a^p \equiv a \pmod p$$

- saves time, instead of multiplying huge numbers millions of times, can use this shortcut
- helps lock and unlock secret codes on the internet

>[!intuition]
Euclidean Algorithm finds the common factor.
Extended GCD backtracks it to solve $ax + by = \gcd$.
Make $\gcd=1$, and BOOM, $x$ becomes your Modular Inverse.
If the mod is prime? Fermat’s Little Theorem lets you skip all of that and just compute $a^{p-2}$.
CP math is just shortcuts built on shortcuts.

| **Concept**                  | **1-Line Formula                                                    | **Key Condition**                      |
| ---------------------------- | ------------------------------------------------------------------- | -------------------------------------- |
| GCD                          | $\gcd(a, b) = \gcd(b, a \bmod b) \quad \text{with } \gcd(a, 0) = a$ | $a, b \ge 0$                           |
| Extended Euclidean Algorithm | $a \cdot x + b \cdot y = \gcd(a, b)$                                | Finds integer coefficients $x, y$      |
| Fermat's Little Theorem      | $a^{p-1} \equiv 1 \pmod p$                                          | $p$ is prime, $\gcd(a, p) = 1$         |
| Modular Inverse              | $a \cdot a^{-1} \equiv 1 \pmod m$                                   | Exists if and only if $\gcd(a, m) = 1$ |