// Computes the high-order half of the 64-bit product.
// Max line length is 57, to fit in hacker.book.
// Derived from Knuth's Algorithm M, altered for signed multiplication.
// Subscript 0 denotes the least significant half (little endian).
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

int mulhs(int u, int v) {
   unsigned u0, v0, w0;
   int u1, v1, w1, w2, t;

   u0 = u & 0xFFFF;  u1 = u >> 16;
   v0 = v & 0xFFFF;  v1 = v >> 16;
   w0 = u0*v0;
   t  = u1*v0 + (w0 >> 16);
   w1 = t & 0xFFFF;
   w2 = t >> 16;
   w1 = u0*v1 + w1;
   return u1*v1 + w2 + (w1 >> 16);
}

int errors;
void error(int u, int v, int rr, int r) {
   errors = errors + 1;
   printf("Error for u = %08x, v = %08x, shd be %08x, got %08x\n", u, v, rr, r);
}

int main() {
   int i, j, n, r;
   long long rr;
   static int test[] = {0, 1, 2, 3, 4, 0xE000, 0xF000, 0xF001,
      0xFFFE, 0xFFFF, 0x10000, 0x10001, 0x10002, 0x20000, 0xE0000000,
      0xE0001000, 0xF0000000, 0xF0001000, 0xF0001001, 0xF0010001,
      0xF0010002, 0xF0020001, 0xF0020002, 0xFFFE0000, 0xFFFE0001,
      0xFFFE0002, 0xFFFF0000, 0xFFFF0001, 0xFFFF0002, 0xFFFFFFFE,
      0xFFFFFFFF};

   n = sizeof(test)/4;

   printf("mulhs:\n");
   for (i = 0; i < n; i += 1) {
      for (j = 0; j < n; j += 1) {
         r = mulhs(test[i], test[j]);
         rr = (long long)test[i]*(long long)test[j];
         rr = rr >> 32;
         if (r != rr) error(test[i], test[j], rr, r);
      }
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n*n);
   return 0;
}
