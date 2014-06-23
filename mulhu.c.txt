// Computes the high-order half of the 64-bit product, unsigned.
// Max line length is 57, to fit in hacker.book. (But not used there.)
// Derived from Knuth's Algorithm M.
// Subscript 0 denotes the least significant half (little endian).
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

// The program below takes 16 ops, 4 of which are multiplies,
// which are of the type unsigned 16 x 16 ==> 32.
// The statement "low = (w1 << 16) + (w0 & 0xFFFF);" placed just before
// the return statement, computes the low-order part in 3 more ops.

unsigned mulhu(unsigned u, unsigned v) {
   unsigned u0, u1, v0, v1, w0, w1, w2, t;

   u0 = u & 0xFFFF;  u1 = u >> 16;
   v0 = v & 0xFFFF;  v1 = v >> 16;
   w0 = u0*v0;
   t  = u1*v0 + (w0 >> 16);
   w1 = t & 0xFFFF;
   w2 = t >> 16;
   w1 = u0*v1 + w1;
   return u1*v1 + w2 + (w1 >> 16);
}

/* The next version does it using only three multiplications.
It is based on:
Let u = a*2**16 + b,
    v = c*2**16 + d.
Then calculate
   p = ac,
   q = bd,
   r = (-a + b)(c - d)
Then uv = p*2**32 + (r + p + q)*2**16 + q.
   There is a difficulty in computing r, because it doesn't
quite fit in a 32-bit word. But because 0 <= a, b, c, d < 2**16,
it is easy to see that
   -2**32 < r < 2**32.
   Thus it can be represented as a 64-bit quantity with the high-order
32 bits being either 0 or all 1's. The low-order 32 bits, rlow, can be
calculated directly from r = (-a + b)*(c - d), using 32-bit
instructions. The high-order 32 bits will be all 1's if the product is
negative, and 0 if it is nonnegative. The product is negative if (-a + b)
and (c - d) have opposite signs. Thus, basically,
   rhigh = ((-a + b) xor (c - d)) >>s 31.
However, if either a = b or c = d, we must ensure that rhigh = 0. It
suffices to test rlow, i.e., follow the above assignment to rhigh with:
   if (rlow == 0) rhigh = 0.
This is because if rlow = 0, it must be the case that either a = b
or c = d, because the product cannot be >= 2**32.
   This leads to the function below.
*/
unsigned mulhu1(unsigned u, unsigned v) {
   unsigned a, b, c, d, p, q, rlow, rhigh;

   a = u >> 16;  b = u & 0xFFFF;
   c = v >> 16;  d = v & 0xFFFF;

   p = a*c;
   q = b*d;
   rlow = (-a + b)*(c - d);
   rhigh = (int)((-a + b)^(c - d)) >> 31;
   if (rlow == 0) rhigh = 0;    // Correction.

   q = q + (q >> 16);   // Overflow cannot occur here.
   rlow = rlow + p;
   if (rlow < p) rhigh = rhigh + 1;
   rlow = rlow + q;
   if (rlow < q) rhigh = rhigh + 1;

   return p + (rlow >> 16) + (rhigh << 16);
}

/* Branch-free version: */

unsigned mulhu2(unsigned u, unsigned v) {
   unsigned a, b, c, d, p, q, x, y, rlow, rhigh, t;

   a = u >> 16;  b = u & 0xFFFF;
   c = v >> 16;  d = v & 0xFFFF;

   p = a*c;
   q = b*d;
   x = -a + b;
   y = c - d;
   rlow = x*y;
   rhigh = (x ^ y) & (rlow | -rlow);
   rhigh = (int)rhigh >> 31;

   q = q + (q >> 16);   // Overflow cannot occur here.
   t = (rlow & 0xFFFF) + (p & 0xFFFF) + (q & 0xFFFF);
   p += (t >> 16) + (rlow >> 16) + (p >> 16) + (q >> 16);
   p += (rhigh << 16);
   return p;
}

int errors;
void error(unsigned u, unsigned v, unsigned rr, unsigned r) {
   errors = errors + 1;
   printf("Error for u = %08x, v = %08x, shd be %08x, got %08x\n", u, v, rr, r);
}

int main() {
   int i, j, n;
   unsigned r;
   unsigned long long rr;
   static unsigned test[] = {0, 1, 2, 3, 4, 0xE000, 0xF000, 0xF001,
      0xFFFE, 0xFFFF, 0x10000, 0x10001, 0x10002, 0x20000, 0xE0000000,
      0xE0001000, 0xF0000000, 0xF0001000, 0xF0001001, 0xF0010001,
      0xF0010002, 0xF0020001, 0xF0020002, 0xFFFE0000, 0xFFFE0001,
      0xFFFE0002, 0xFFFF0000, 0xFFFF0001, 0xFFFF0002, 0xFFFFFFFE,
      0xFFFFFFFF};

   n = sizeof(test)/4;

   printf("mulhu:\n");
   for (i = 0; i < n; i += 1) {
      for (j = 0; j < n; j += 1) {
         r = mulhu(test[i], test[j]);
         rr = (unsigned long long)test[i]*(unsigned long long)test[j];
         rr = rr >> 32;
         if (r != rr) error(test[i], test[j], (unsigned)rr, r);
      }
   }

   printf("mulhu1:\n");
   for (i = 0; i < n; i += 1) {
      for (j = 0; j < n; j += 1) {
         r = mulhu1(test[i], test[j]);
         rr = (unsigned long long)test[i]*(unsigned long long)test[j];
         rr = rr >> 32;
         if (r != rr) error(test[i], test[j], (unsigned)rr, r);
      }
   }

   printf("mulhu2:\n");
   for (i = 0; i < n; i += 1) {
      for (j = 0; j < n; j += 1) {
         r = mulhu2(test[i], test[j]);
         rr = (unsigned long long)test[i]*(unsigned long long)test[j];
         rr = rr >> 32;
         if (r != rr) error(test[i], test[j], (unsigned)rr, r);
      }
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n*n);
   return 0;
}
