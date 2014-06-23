/* Divide signed by various constants d by multiplying by an
approximate reciprocal of d, computing the remainder, and correcting by
adding r/d.  Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

/* The code below uses the code on pp. 207-208 and Figure 8-2 on p. 174
to do the multiply high signed. 16 ops, including 4 multiplies. This
code is not in the book because it's very similar to the unsigned
version and because the following procedure (divs3b) is better. */

int divs3a(int n) {
   int n0, n1, w0, w1, w2, t, q;

   n0 = n & 0xFFFF;
   n1 = n >> 16;
   w0 = n0*0x5556;
   t  = n1*0x5556 + (w0 >> 16);
   w1 = t & 0xFFFF;
   w2 = t >> 16;
   w1 = n0*0x5555 + w1;
   q = n1*0x5555 + w2 + (w1 >> 16);
   return q - (n >> 31);        // Add 1 if n < 0.
}

/* The code below multiplies by 1/3 (binary 0.010101...) by a series of
shifts right and adds. Each step introduces an error, due to bits being
shifted off the right end of the register. The error is corrected at the
end, by computing the remainder r with respect to the approximate
quotient, and adding r/3. This method is adapted from a suggestion by
GLS for unsigned division by a constant.
   This takes 17 ops including two multiplies. If the multiply by 3 is
done by a shift and add, and alternative 2 (or alternative 1 with the
multiply by 5 changed to a shift and add) is used, it is 20 elementary
instructions. Alternative 2 has a little instruction-level parallelism.
It was found by Aha!.
   A more accurate quotient estimate can be obtained by changing the
first executable line to

                        q = (n >> 1) + (n >> 3);

(which makes q too large by a factor of 2, but it has one more bit of
accuracy), and then inserting just before the assignment to r,

                              q = q >> 1;

With this variation, the remainder is at most 8. However, there does not
seem to be any better code for calculating r/3 with r limited to 8, than
there is for r limited to 14 (4 elementary ops in either case). Thus
using the idea would cost one instruction. */

int divs3b(int n) {
   int q, r;

   n = n + (n>>31 & 2);         // Add 2 if n < 0.
   q = (n >> 2) + (n >> 4);     // q = n*0.0101 (approx).
   q = q + (q >> 4);            // q = n*0.01010101.
   q = q + (q >> 8);
   q = q + (q >> 16);
   r = n - q*3;                 // 0 <= r <= 14.
   if (r < 0) printf("Negative r = %d\n", r);
   return q + (11*r >> 5);      // Returning q + r/3.
// return q + (5*(r + 1) >> 4);         // Alternative 1.
// return q + ((r + 5 + (r << 2)) >> 4);// Alternative 2.
}

/* The code below multiplies by 1/5 (binary 0.00110011...) by the shift
right and add, and then correct, method. 17 ops, including two
multiplies, or 21 elementary operations. 0 <= r <= 24. Not in book. */

int divs5a(int n) {
   int q, r;

   n = n + (n>>31 & 4);         // Add 4 if n < 0.
   q = (n >> 3) + (n >> 4);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   r = n - q*5;
   return q + (13*r >> 6);
}

/* Code below is 18 ops including two multiplies, or 20 elementary ops.
0 <= r <= 9. */

int divs5b(int n) {
   int q, r;

   n = n + (n>>31 & 4);
   q = (n >> 1) + (n >> 2);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   q = q >> 2;
   r = n - q*5;
   return q + (7*r >> 5);
// return q + (r>4) + (r>9);
}

/* The code below takes 17 ops including two multiplies, or 22
elementary ops. 0 <= r <= 29. The 11*r >> 6 can be done in 5 ops.
According to Aha! there are no 4-instruction solutions for this
(computing r/6 for 0 <= r <= 29), although there may be other
5-instruction solutions. Not in book. */

int divs6a(int n) {
   int q, r;

   n = n + (n>>31 & 5);
   q = (n >> 3) + (n >> 5);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   r = n - q*6;
   return q + (11*r >> 6);
}

/* The variation below computes q to two more bits of accuracy than does
divs6a. 0 <= r <= 11. However, the approximation of 3r/16 for r/6 is not
quite good enough; we still have to use 11r/64. Hence this is worse than
divs6a by one instruction. However, the correction to q is at most 1,
which gives two interesting alternatives. If the multiplication by 6 is
done by shifts and adds (3 instructions), alternatives 1 and 2 are 17
and 16 elementary instructions, resp. These alternatives are useful
whenever the approximate quotient is off by at most 1 (i.e., 0 <= r <
2d). 18 ops including one multiply, or 20 elementary ops. */

int divs6b(int n) {
   int q, r;

   n = n + (n>>31 & 5);
   q = (n >> 1) + (n >> 3);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   q = q >> 2;
   r = n - q*6;
   return q + ((r + 2) >> 3);
// return q + (r > 5);
}

/* Code below is 17 ops including two multiplies, or 21 elementary ops.
0 <= r <= 34. Not in book. */

int divs7a(int n) {
   int q, r;

   n = n + (n>>31 & 6);
   q = (n >> 3) + (n >> 6);   // q = n*0.001001 (approx).
   q = q + (q >> 6);          // q = n*0.001001001001.
   q = q + (q >> 12);
   q = q + (q >> 24);
   r = n - q*7;               // 0 <= r <= 34.
   return q + (37*r >> 8);    // Returning q + r/7.
}

/* Code below is 18 ops, including one multiply, or 19 elementary ops.
Uses 4/7 = 0.1001 0010 0100 1001 0010 0100 1001 0010. 0 <= r <= 13. */

int divs7b(int n) {
   int q, r;

   n = n + (n>>31 & 6);
   q = (n >> 1) + (n >> 4);
   q = q + (q >> 6);
   q = q + (q>>12) + (q>>24);
   q = q >> 2;
   r = n - q*7;
   return q + ((r + 1) >> 3);
// return q + (r > 6);
}

/* Code below is uses 8/9 = 0.1110 0011 1000 1110 0011 1000 1110 0011.
20 ops including the multiply, or 21 elementary ops. 0 <= r <= 13. */

int divs9(int n) {
   int q, r;

   n = n + (n>>31 & 8);
   q = (n >> 1) + (n >> 2) +
       (n >> 3);
   q = q + (q >> 6);
   q = q + (q>>12) + (q>>24);
   q = q >> 3;
   r = n - q*9;
   return q + ((r + 7) >> 4);
// return q + (r > 8);
}

/* Code below uses 8/10 = 0.1100 1100 1100 1100 1100 1100 1100 1100.
18 ops including the multiply, or 20 elementary ops. 0 <= r <= 13. */

int divs10(int n) {
   int q, r;

   n = n + (n>>31 & 9);
   q = (n >> 1) + (n >> 2);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   q = q >> 3;
   r = n - q*10;
   return q + ((r + 6) >> 4);
// return q + (r > 9);
}

/* Code below uses 8/11 = 0.1011 1010 0010 1110 1000 1011 1010 0010.
20 ops including the multiply, or 23 elementary ops. 0 <= r <= 15. */

int divs11(int n) {
   int q, r;

   n = n + (n>>31 & 10);
   q = (n >> 1) + (n >> 2) -
       (n >> 5) + (n >> 7);
   q = q + (q >> 10);
   q = q + (q >> 20);
   q = q >> 3;
   r = n - q*11;
   return q + ((r + 5) >> 4);
// return q + (r > 10);
}

/* Code below uses 8/12 = 0.1010 1010 1010 1010 1010 1010 1010 1010.
18 ops including the multiply, or 20 elementary ops. 0 <= r <= 15. */

int divs12(int n) {
   int q, r;

   n = n + (n>>31 & 11);
   q = (n >> 1) + (n >> 3);
   q = q + (q >> 4);
   q = q + (q >> 8);
   q = q + (q >> 16);
   q = q >> 3;
   r = n - q*12;
   return q + ((r + 4) >> 4);
// return q + (r > 11);
}

/* Code below uses 8/13 = 0.1001 1101 1000 1001 1101 1000 1001 1101.
20 ops including the multiply, or 23 elementary ops. 0 <= r <= 20. */

int divs13(int n) {
   int q, r;

   n = n + (n>>31 & 12);
   q = (n>>1) + (n>>4);
   q = q + (q>>4) + (q>>5);
   q = q + (q>>12) + (q>>24);
   q = q >> 3;
   r = n - q*13;
   return q + ((r + 3) >> 4);
// return q + (r > 12);
}

/* Code below uses 64/100 = 0.1010 0011 1101 0111 0000 1010 0011 1101.
24 ops including the multiply, or 28 elementary ops. 0 <= r <= 105. */

int divs100(int n) {
   int q, r;

   n = n + (n>>31 & 99);
   q = (n >> 1) + (n >> 3) + (n >> 6) - (n >> 10) +
       (n >> 12) + (n >> 13) - (n >> 16);
   q = q + (q >> 20);
   q = q >> 6;
   r = n - q*100;
   return q + ((r + 28) >> 7);
// return q + (r > 99);
}

/* Code below uses 1/1000 = 0.0000 0000 0100 0001 1000 1001 0011 0111.
23 ops including two multiplies, or 32 elementary ops. 0 <= r <= 5997.
Not in book. */

int divs1000a(int n) {
   int q, r, t;

   n = n + (n>>31 & 999);
   t = n - (n >> 6) - (n >> 3);
   q = (n >> 10) + (n >> 16) + (n >> 17) + (n >> 21) + (n >> 24) +
       (t >> 26);
   r = n - q*1000;            // 0 <= r <= 5997.
   return q + (4195*r >> 22); // Returning q + r/1000.
// {int t = r + (r << 1);     // Alternative showing
// t = t + (t << 5);          // expansion of multipli-
// t = t + (r << 12);         // cation by 4195.
// return q + (t >> 22);}
}

/* Code below uses 512/1000 = 0.1000 0011 0001 0010 0110 1110 1001 0111.
26 ops including one multiply, or 30 elementary ops. 0 <= r <= 1015. */

int divs1000b(int n) {
   int q, r, t;

   n = n + (n>>31 & 999);
   t = (n >> 7) + (n >> 8) + (n >> 12);
   q = (n >> 1) + t + (n >> 15) + (t >> 11) + (t >> 14) +
       (n >> 26) + (t >> 21);
   q = q >> 9;
   r = n - q*1000;
   return q + ((r + 24) >> 10);
// return q + (r > 999);
}

int errors;
void error(int n, int r) {
   errors = errors + 1;
   printf("Error for n = %08x, got %08x (%d dec)\n", n, r, r);
}

int main() {
   int n;
   int q;
   /* Make low and high equal for an exhaustive test. */
   const int low = -15000000, high = -15000000;

   printf("divs3a:\n");
   n = low; do {
      q = divs3a(n);
      if (q != n/3) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs3b:\n");
   n = low; do {
      q = divs3b(n);
      if (q != n/3) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs5a:\n");
   n = low; do {
      q = divs5a(n);
      if (q != n/5) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs5b:\n");
   n = low; do {
      q = divs5b(n);
      if (q != n/5) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs6a:\n");
   n = low; do {
      q = divs6a(n);
      if (q != n/6) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs6b:\n");
   n = low; do {
      q = divs6b(n);
      if (q != n/6) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs7a:\n");
   n = low; do {
      q = divs7a(n);
      if (q != n/7) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs7b:\n");
   n = low; do {
      q = divs7b(n);
      if (q != n/7) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs9:\n");
   n = low; do {
      q = divs9(n);
      if (q != n/9) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs10:\n");
   n = low; do {
      q = divs10(n);
      if (q != n/10) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs11:\n");
   n = low; do {
      q = divs11(n);
      if (q != n/11) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs12:\n");
   n = low; do {
      q = divs12(n);
      if (q != n/12) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs13:\n");
   n = low; do {
      q = divs13(n);
      if (q != n/13) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs100:\n");
   n = low; do {
      q = divs100(n);
      if (q != n/100) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs1000a:\n");
   n = low; do {
      q = divs1000a(n);
      if (q != n/1000) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs1000b:\n");
   n = low; do {
      q = divs1000b(n);
      if (q != n/1000) error(n, q);
      n = n + 1;
   } while (n != high);

   if (errors == 0)
      printf("Passed all tests.\n");
}
