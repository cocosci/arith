# arith
An example of the [arithmetic coding](https://en.wikipedia.org/wiki/Arithmetic_coding) compression scheme written in
Python. 

## How arithmetic coding works

Arithmetic coding is a way to encode a list of symbols into a single fraction _f_, where 0 ≤ _f_ < 1.

## A quick example.

Say, for example, we want to encode the string "BANANA!". (For this example, the "!" represents an internal
"end-of-data" symbol.)

First, we build a frequency table of symbols:

* `A` occurs 3 times.
* `N` occurs 2 times.
* `!` occurs 1 time.
* `B` occurs 1 time.

Using this frequency table, we build a list of ranges representing the cumulative frequency of a symbol occurring:

* `A` starts at 0/7 with a probability of 3/7.
* `N` starts at 3/7 with a probability of 2/7.
* `!` starts at 5/7 with a probability of 1/7.
* `B` starts at 6/7 with a probability of 1/7.

With this cumulative frequency table, we begin computing the fraction and width:

1. Initialize the fraction at **0/7** and the width at **7/7**.
2. `B` starts at 6/7 with a probability of 1/7. Add 6/7 of the width to the fraction and multiply the width by 1/7.
   Fraction is now **6/7** and width is now **1/7**.
3. `A` starts at 0/7 with a probability of 3/7. Add 0/7 of the width to the fraction and multiply the width by 3/7.
   Fraction is now **42/49** and width is now **3/49**.
4. `N` starts at 3/7 with a probability of 2/7. Add 3/7 of the width to the fraction and multiply the width by 2/7.
   Fraction is now **303/343** and width is now **6/343**.
5. `A` starts at 0/7 with a probability of 3/7. Add 0/7 of the width to the fraction and multiply the width by 3/7.
   Fraction is now **2121/2401** and width is now **18/2401**.
6. `N` starts at 3/7 with a probability of 2/7. Add 3/7 of the width to the fraction and multiply the width by 2/7.
   Fraction is now **14901/16807** and width is **36/16807**.
7. `A` starts at 0/7 with a probability of 3/7. Add 0/7 of the width to the fraction and multiply the width by 3/7.
   Fraction is now **104307/117649** and width is **108/117649**.
8. `!` starts at 5/7 with a probability of 1/7. Add 5/7 of the width to the fraction and multiply the width by 1/7.
   Fraction is now **730689/823543** and width is **108/823543**.

The string "BANANA!" is now encoded as a fraction **730689/823543** and a width of **730797/823543**.
In other words, **730689/823543** ≤ "BANANA!" < **730797/823543**.

We could use this range as it exists by encoding the numerator and denominator in binary:

* 730689 = 10110010011001000001
* 823543 = 11001001000011110111

Unfortunately, that representation uses 40 bits. There's a better way, though: find a fraction in the range that can be
represented in the form **N/(2^B)**, where **N** is an integer, **^** is the exponent symbol and **B** is the number of
bits.

We iterate through each possible value of B and compute an approximate integer N through each step:

1. Does **1/(2^0)** fit in our range? (Nope, keep going.)
2. Does **2/(2^1)** fit in our range? (Nope, keep going.)
3. Does **4/(2^2)** fit in our range? (Nope, keep going.)
4. Does **8/(2^3)** fit in our range? (Nope, keep going.)
5. Does **15/(2^4)** fit in our range? (Nope, keep going.)
6. Does **29/(2^5)** fit in our range? (Nope, keep going.)
7. Does **57/(2^6)** fit in our range? (Nope, keep going.)
8. Does **114/(2^7)** fit in our range? (Nope, keep going.)
9. Does **228/(2^8)** fit in our range? (Nope, keep going.)
10. Does **455/(2^9)** fit in our range? (Nope, keep going.)
11. Does **909/(2^10)** fit in our range? (Nope, keep going.)
12. Does **1818/(2^11)** fit in our range? (Nope, keep going.)
13. Does **3635/(2^12)** fit in our range? (Nope, keep going.)
14. Does **7269/(2^13)** fit in our range? Yes!

Now we have the fraction **7269/8192**. We can encode this fraction much simpler in binary:

* 7269 = 01110001100101
* 8192 = 10000000000000

Internally, we can deduce the denominator by the number of bits in the numerator, so we can simplify the fraction even
further:

* **7269/8192** = 1110001100101

We deduce the denominator by counting the number of bits in the numerator and use that bit count to compute **2^N**.

Compare our original string in bits to the new fraction in bits:

* "BANANA!" = 01000010010000010100111001000001010011100100000100100001
* **7269/8192** = 1110001100101

With arithmetic coding, we reduced the string from 56 bits to 13 bits — a 77% reduction in space.
