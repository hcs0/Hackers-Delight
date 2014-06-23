// Computes pop(x) - pop(y).
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

int pop(unsigned x) {           // Alg. of Fig. 5-2. */
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x >> 8);
   x = x + (x >> 16);
   return x & 0x0000003F;
}

int popDiff(unsigned x, unsigned y) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   y = ~y;
   y = y - ((y >> 1) & 0x55555555);
   y = (y & 0x33333333) + ((y >> 2) & 0x33333333);
   x = x + y;
   x = (x & 0x0F0F0F0F) + ((x >> 4) & 0x0F0F0F0F);
   x = x + (x >> 8);
   x = x + (x >> 16);
   return (x & 0x0000007F) - 32;
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, y = %08x, popDiff = %d\n",
      x, y, popDiff(x, y));
}

int main() {
   int i, j, n;
   unsigned x, y;
   static unsigned int test[] = {0, 1, 2, 3, 4, 5, 6, 7,
      8, 9, 10, 11, 12, 13, 14, 15, 16, 17,
      0x3F, 0x40, 0x41, 0x7f, 0x80, 0x81, 0xfe, 0xff,
      0x4000, 0x4001, 0x7000, 0x7fff,
      0x55555555, 0xAAAAAAAA, 0xFF000000, 0xC0C0C0C0,
      0x0FFFFFF0, 0x80000000, 0xFFFFFFFE, 0xFFFFFFFF};

   n = sizeof(test)/4;

   for (i = 0; i < n; i ++) {
      x = test[i];
      for (j = 0; j < n; j ++) {
         y = test[j];
         if (popDiff(x, y) != pop(x) - pop(y)) error(x, y);
      }
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n*n);
}
