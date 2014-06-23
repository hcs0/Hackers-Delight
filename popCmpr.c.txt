// Determines which is larger, pop(x) or pop(y).
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

int popCmpr(unsigned xp, unsigned yp) {
   unsigned x, y;
   x = xp & ~yp;                // Clear bits where
   y = yp & ~xp;                // both are 1.
   while (1) {
      if (x == 0) return y | -y;
      if (y == 0) return 1;
      x = x & (x - 1);          // Clear one bit
      y = y & (y - 1);          // from each.
   }
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, y = %08x, popCmpr = %d\n",
      x, y, popCmpr(x, y));
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
         if (pop(x) > pop(y) && popCmpr(x, y) > 0 ||
             pop(x) < pop(y) && popCmpr(x, y) < 0 ||
             pop(x) == pop(y) && popCmpr(x, y) == 0) ;
         else error(x, y);
      }
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n*n);
}
