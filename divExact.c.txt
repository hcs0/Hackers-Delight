/* This is a few experiments in computing the n/d by first computing
rem(n, d), and then subtracting that from n. The division is then an
exact division, so the quotient can be obtained by multiplication by the
inverse of d. That is,

                     n/d = (n - rem(n, d))*dbar.

This material is in HD second edition, but not in the first. */

#include <stdio.h>

/* The code below is 11 ops, including 2 multiplications.
(The constant 0xAAAAAAAB can generate 0x55555555 in one instruction). */

unsigned divu3a(unsigned n) {
   unsigned r;

   r = (0x55555555*n + (n >> 1) - (n >> 3)) >> 30;
   return (n - r)*0xAAAAAAAB;
}

/* This is the above code with the multiplications expanded.
24 elementary ops. */

unsigned divu3b(unsigned n) {
   unsigned r;

   r = n + (n << 2);
   r = r + (r << 4);
   r = r + (r << 8);
   r = r + (r << 16);
   r = r + ((n >> 1) - (n >> 3)) >> 30;
   n = n - r;                   // Next mult. by inv(3).
   r = (n << 1) + (n << 3);     // * 0xA.
   r = r + (r << 4);            // * 0xAA.
   r = r + (r << 8);            // * 0xAAAA.
   r = r + (r << 16);           // * 0xAAAAAAAA.
   return r + n;                // * 0xAAAAAAAB.
}

/* The code below is 15 ops, including two multiplications. */

int divs3(int n) {
   unsigned r;

   r = n;
   r = (0x55555555*r + (r >> 1) - (r >> 3)) >> 30;
   r = r - (((unsigned)n >> 31) << (r & 2));
   return (n - r)*0xAAAAAAAB;
}

/* The code below is 12 ops, including two multiplications, plus an
indexed load. The constant 0x19999999 can be obtained by shifting
0xCCCCCCCD right 3 positions. */

unsigned divu10(unsigned n) {
   unsigned r;
   static char table[16] = {0, 1, 2, 2, 3, 3, 4, 5,
                            5, 6, 7, 7, 8, 8, 9, 0};

   r = (0x19999999*n + (n >> 1) + (n >> 3)) >> 28;
   r = table[r];
   return ((n - r) >> 1)*0xCCCCCCCD;
}

int errors;
void error(unsigned n, unsigned q) {
   errors = errors + 1;
   printf("Error for n = %08x, got %08x (%d dec)\n", n, q, q);
}

int main() {
   int n;
   unsigned q;
   /* Make low and high equal for an exhaustive test. */
   const int low = -15000000, high = -15000000;

   printf("divu3a:\n");
   n = low; do {
      q = divu3a(n);
      if (q != (unsigned)n/3) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divu3b:\n");
   n = low; do {
      q = divu3b(n);
      if (q != (unsigned)n/3) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divs3:\n");
   n = low; do {
      q = divs3(n);
      if (q != n/3) error(n, q);
      n = n + 1;
   } while (n != high);

   printf("divu10:\n");
   n = low; do {
      q = divu10(n);
      if (q != (unsigned)n/10) error(n, q);
      n = n + 1;
   } while (n != high);

   if (errors == 0)
      printf("Passed all tests.\n");
}
