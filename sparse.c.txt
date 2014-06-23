#include <stdlib.h>
#include <stdio.h>
/* For an array with elements 0, 2, 32, 47, 48, and 95 defined: */
unsigned bits[3] = {0x00000005, 0x00018001, 0x80000000};
int bitsum[3] = {0, 2, 5};

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

int main(int argc, char *argv[]) {
   int i, j, k, sparse_i;
   unsigned mask;

   i = strtol(argv[1], NULL, 10);
// ------------------------------ cut ----------------------------------
   j = i >> 5;                  // j = i/32.
   k = i & 31;                  // k = rem(i, 32);
   mask = 1 << k;               // A "1" at position k.
   if ((bits[j] & mask) == 0) goto no_such_element;
   mask = mask - 1;             // 1's to right of k.
   sparse_i = bitsum[j] + pop(bits[j] & mask);
// ---------------------------- end cut --------------------------------
   printf("sparse_i = %d.\n", sparse_i);
   return 0;
no_such_element:
   printf("No such element.\n");
   return 1;
}
