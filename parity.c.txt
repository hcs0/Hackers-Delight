// Programs for computing the parity of a word.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

int parity1(unsigned x) {
   unsigned y;

   y = x ^ (x >> 1);
   y = y ^ (y >> 2);
   y = y ^ (y >> 4);
   y = y ^ (y >> 8);
   y = y ^ (y >>16);
   return y & 1;
}

int parity1a(unsigned x) {
   unsigned y;

   y = x ^ (x >>16);
   y = y ^ (y >> 8);
   y = y ^ (y >> 4);
   y = 0x6996 >> (y & 0xF);     // Falk Hueffner's trick.
   return y & 1;
}

int parity2(unsigned x) {
   int p;

   x = x ^ (x >> 1);
   x = (x ^ (x >> 2)) & 0x11111111;
   x = x*0x11111111;
   p = (x >> 28) & 1;
   return p;
}

int parity3(unsigned x) {
   unsigned y;

   y = (x*0x10204081) & 0x888888FF;
   return (y%1920) & 0xFF;      // Returns a byte with even parity.
}

int parity4(unsigned x) {
   unsigned y;

   y = (x*0x00204081) | 0x3DB6DB00;
   y = (y%1152) & 0xFF;
   return y ^ 0x80;             // Change to even parity so test2 can be used.
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, r, n;
   static unsigned test[] = {0,0, 1,1, 2,1, 3,0, 4,1, 5,0,
      6,0, 7,1, 8,1, 9,0, 10,0, 11,1, 12,0, 13,1, 14,1,
      15,0, 16,1, 17,0, 18,0, 19,1, 20,0, 21,1, 22,1, 23,0,
      24,0, 25,1, 26,1, 27,0, 28,1, 29,0, 30,0, 31,1,
      0x55555555,0, 0xAAAAAAAA,0, 0x77777770,1,
      0x80000000,1, 0x80000001,0, 0xFFFFFFFE,1, 0xFFFFFFFF,0};
   static unsigned test2[] = {0,0, 1,0x81, 2,0x82, 3,3, 4,0x84,
      5,5, 6,6, 7,0x87, 8,0x88, 9,9, 10,10, 11,0x8B, 12,12,
      13,0x8D, 14,0x8E, 15,15, 16,0x90, 0x7E,0x7E, 0x7F,0xFF};

   n = sizeof(test)/4;

   printf("parity1:\n");
   for (i = 0; i < n; i += 2) {
      r = parity1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("parity1a:\n");
   for (i = 0; i < n; i += 2) {
      r = parity1a(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("parity2:\n");
   for (i = 0; i < n; i += 2) {
      r = parity2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);

   n = sizeof(test2)/4;

   printf("parity3:\n");
   for (i = 0; i < n; i += 2) {
      r = parity3(test2[i]);
      if (r != test2[i+1]) error(test2[i], r);}

   printf("parity4:\n");
   for (i = 0; i < n; i += 2) {
      r = parity4(test2[i]);
      if (r != test2[i+1]) error(test2[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);
}
