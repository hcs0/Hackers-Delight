// Program for computing the integer cube root.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

// Execution time is 3 + (11 + mul)11 = 124 + 11*mul (avg) cycles.
// ------------------------------ cut ----------------------------------
int icbrt1(unsigned x) {
   int s;
   unsigned y, b;

   y = 0;
   for (s = 30; s >= 0; s = s - 3) {
      y = 2*y;
      b = (3*y*(y + 1) + 1) << s;
      if (x >= b) {
         x = x - b;
         y = y + 1;
      }
   }
   return y;
}
// ---------------------------- end cut --------------------------------

// Strength reduced to avoid a multiplication of variables.
// Execution time is 4 + 13.5*11 = 152 (avg) cycles.

int icbrt2(unsigned x) {
   int s;
   unsigned y, b, y2;

   y2 = 0;
   y = 0;
   for (s = 30; s >= 0; s = s - 3) {
      y2 = 4*y2;
      y = 2*y;
      b = (3*(y2 + y) + 1) << s;
      if (x >= b) {
         x = x - b;
         y2 = y2 + 2*y + 1;
         y = y + 1;
      }
   }
   return y;
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, n;
   unsigned r;
   static unsigned test[] = {0,0, 1,1, 2,1, 3,1, 4,1, 5,1,
      6,1, 7,1, 8,2, 9,2, 10,2, 11,2, 12,2, 13,2, 14,2,
      15,2, 16,2, 17,2, 18,2, 19,2, 20,2, 21,2, 22,2, 23,2,
      24,2, 25,2, 26,2, 27,3, 28,3, 29,3, 30,3, 31,3,
      32,3, 33,3, 34,3, 35,3, 36,3, 37,3, 38,3, 39,3, 40,3,
      99,4, 100,4, 101,4, 32767,31, 32768,32, 32769,32,
      1073741823,1023, 1073741824,1024, 1073741825,1024,
      0x80000000,1290, 0xFFFFFFFF,1625};

   n = sizeof(test)/4;

   printf("icbrt1:\n");
   for (i = 0; i < n; i += 2) {
      r = icbrt1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("icbrt2:\n");
   for (i = 0; i < n; i += 2) {
      r = icbrt2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
   return errors;
}
