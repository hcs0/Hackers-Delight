// Program for computing x**n for integers.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

int iexp(int x, unsigned n) {
   int p, y;

   y = 1;                     // Initialize result
   p = x;                     // and p.
   while(1) {
      if (n & 1) y = p*y;     // If n is odd, mult by p.
      n = n >> 1;             // Position next bit of n.
      if (n == 0) return y;   // If no more bits in n.
      p = p*p;                // Power for next bit of n.
   }
}

int errors;
void error(int x, int y, int r) {
   errors = errors + 1;
   printf("Error for x = %08x, y = %08x, got %d\n", x, y, r);
}

int main() {
   int i, r, n;
   static int test[] = {0,0,1, 0,1,0, 1,0,1, 0,2,0,
      1,1,1, 2,0,1, 0,3,0, 1,2,1, 2,1,2, 3,0,1,
      0,4,0, 1,3,1, 2,2,4, 3,1,3, 4,0,1,
      0,5,0, 1,4,1, 2,3,8, 3,2,9, 4,1,4, 5,0,1,
      0,6,0, 1,5,1, 2,4,16, 3,3,27, 4,2,16, 5,1,5, 6,0,1,
      0,7,0, 1,6,1, 2,5,32, 3,4,81, 4,3,64, 5,2,25, 6,1,6, 7,0,1,
      2,6,64, 2,7,128, 2,8,256, 2,9,512, 2,10,1024, 2,11,2048,
      2,12,4096, 2,13,8192, 2,14,16384, 2,15,32768, 2,16,65536,
      2,30,1073741824, 2,31,0x80000000, 2,32,0,
      3,13,1594323, 4,4,256, 5,5,3125, 6,6,46656, 7,7,823543,
      8,8,16777216, 9,9,387420489, 10,10,1410065408,
      0,0xFFFFFFFF,0, 1,0xFFFFFFFF,1, 2,0xFFFFFFFF,0,
      -1,0,1, -2,0,1, -2,1,-2, -2,2,4, -2,3,-8, -2,4,16, -2,5,-32,
      -3,0,1, -3,1,-3, -3,2,9, -3,3,-27, -3,4,81, -3,5,-243};

   n = sizeof(test)/4;

   printf("iexp:\n");
   for (i = 0; i < n; i += 3) {
      r = iexp(test[i], test[i+1]);
      if (r != test[i+2]) error(test[i], test[i+1], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/3);
}
