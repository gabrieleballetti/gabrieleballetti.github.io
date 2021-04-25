# The IEEE standard

# Negative and positive zeroes





Single precision: 36 bits, organized as a 1-bit sign, an 8-bit exponent, and a 27-bit significand.
Double precision: 72 bits, organized as a 1-bit sign, an 11-bit exponent, and a 60-bit significand.

[The largest value representable by an n bit integer is 2n-1. As noted above, a float has 24 bits of precision in the significand which would seem to imply that 224 wouldn't fit.

However.]

Smallest non representable integer
2mantissa bits + 1 + 1

The +1 in the exponent (mantissa bits + 1) is because, if the mantissa contains abcdef... the number it represents is actually 1.abcdef... × 2^e, providing an extra implicit bit of precision.

Therefore, the first integer that cannot be accurately represented and will be rounded is:
For float, 16,777,217 (224 + 1).
For double, 9,007,199,254,740,993 (253 + 1).

For many applications, the problem of floating-point errors can be attenuated by cleverly re-arranging the order in which certain operations are performed. Since the main culprit is adding large numbers with small ones together, a common technique is to add elements “in order”. Adding up the smaller numbers first allows keeping as much precision as possible. Many different techniques have also been proposed to this problem, such as the pairwise summation or the compensated summation (also known as the “Kahan summation algorithm”).

# POSITIVE AND NEGATIVE 0

According to the standard, negative zero exists but it is equal to positive zero. For almost all purposes, the two behave the same way and many consider the existence of a negative to be an implementation detail. There are, however, some functions that behave quite differently, namely division and atan2:

#include <math.h>
#include <stdio.h>

int main() {
    double x = 0.0;
    double y = -0.0;
    printf("%.08f == %.08f: %d\n", x, y, x == y);
    printf("%.08f == %.08f: %d\n", 1 / x, 1 / y, 1 / x == 1 / y);
    printf("%.08f == %.08f: %d\n", atan2(x, y), atan2(y, y), atan2(x, y) == atan2(y, y));
}
The result from this code is:

0.00000000 == -0.00000000: 1
1.#INF0000 == -1.#INF0000: 0
3.14159265 == -3.14159265: 0