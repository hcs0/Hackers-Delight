// Divide long unsigned by hardware "restoring" method.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

// ------------------------------ cut ----------------------------------
unsigned divluh(unsigned x, unsigned y, unsigned z) {
   // Divides (x || y) by z.
   int i;
   unsigned t;

   for (i = 1; i <= 32; i++) {
      t = (int)x >> 31;         // All 1's if x(31) = 1.
      x = (x << 1) | (y >> 31); // Shift x || y left
      y = y << 1;               // one bit.
      if ((x | t) >= z) {
         x = x - z;
         y = y + 1;
      }
   }
   return y;                    // Remainder is x.
}
// ------------------------------ cut ----------------------------------

// Version based on nonrestoring hardware algorithm:
// This more-or-less verifies the algorithm given in HD,
// but we don't include this C version because it's not very efficient.
unsigned divluh1(unsigned x, unsigned y, unsigned z) {
   // Divides (x || y) by z.
   int i;
   unsigned c;

   c = 0;
   for (i = 1; i <= 32; i++) {
      if (c == 0) {
         c = x >> 31;
         x = (x << 1) | (y >> 31); // Shift x || y left
         y = y << 1;               // one bit.
         c = c ^ (x < z);
         x = x - z;
      }
      else {
         c = x >> 31;
         x = (x << 1) | (y >> 31); // Shift x || y left
         y = y << 1;               // one bit.
         x = x + z;
         c = c ^ (x < z);
      }
      y = y + (1 - c);
   }
   return y;                    // Remainder is x.
}

int errors;
void error(unsigned x, unsigned y, unsigned z, unsigned r) {
   errors = errors + 1;
   printf("Error for x = %08x, y = %08x, z = %08x,"
      " got %08x (%d dec)\n", x, y, z, r, r);
}

int main() {
   int i, r, n;
   static unsigned test[] = {0,0,1,0, 0,1,1,1, 0,21,7,3,
   0,0xffffffff,1,0xffffffff, 0,0xffffffff,10,0x19999999,
   1,1,2,0x80000000, 1,1,3,0x55555555, 2,4,4,0x80000001,
   0x32123456,0x789abcde,0x45600000,0xb8c45446,
   0x80000000,0x00000000,0x80000001,0xfffffffe,
   0xfffffffe,0xffffffff,0xffffffff,0xffffffff,
   0x40000000,0x00000000,0xc0000000,0x55555555,
   };

   n = sizeof(test)/4;

   printf("divluh:\n");
   for (i = 0; i < n; i += 4) {
      r = divluh(test[i], test[i+1], test[i+2]);
      if (r != test[i+3]) error(test[i], test[i+1], test[i+2], r);
   }

   printf("divluh1:\n");
   for (i = 0; i < n; i += 4) {
      r = divluh1(test[i], test[i+1], test[i+2]);
      if (r != test[i+3]) error(test[i], test[i+1], test[i+2], r);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n/4);
}
