/* Determines which of the 256 Boolean functions of
three variables can be implemented with three binary
Boolean instructions if the instruction set includes
all 16 binary Boolean operations. */

#include <stdio.h>

char found[256];

unsigned char boole(int op, unsigned char x,
                            unsigned char y) {
   switch (op) {
      case  0: return 0;
      case  1: return x & y;
      case  2: return x & ~y;
      case  3: return x;
      case  4: return ~x & y;
      case  5: return y;
      case  6: return x ^ y;
      case  7: return x | y;
      case  8: return ~(x | y);
      case  9: return ~(x ^ y);
      case 10: return ~y;
      case 11: return x | ~y;
      case 12: return ~x;
      case 13: return ~x | y;
      case 14: return ~(x & y);
      case 15: return 0xFF;
   }
}

#define NB 16           // Number of Boolean operations.

int main() {

   int i, j, o1, i1, i2, o2, j1, j2, o3, k1, k2;
   unsigned char fun[6];// Truth table, 3 columns for
                        // x, y, and z, and 3 columns
                        // for computed functions.

   fun[0] = 0x0F;       // Truth table column for x,
   fun[1] = 0x33;       // y,
   fun[2] = 0x55;       // and z.

   for (o1 = 0; o1 < NB; o1++) {
   for (i1 = 0; i1 < 3; i1++) {
   for (i2 = 0; i2 < 3; i2++) {
      fun[3] = boole(o1, fun[i1], fun[i2]);
      for (o2 = 0; o2 < NB; o2++) {
      for (j1 = 0; j1 < 4; j1++) {
      for (j2 = 0; j2 < 4; j2++) {
         fun[4] = boole(o2, fun[j1], fun[j2]);
         for (o3 = 0; o3 < NB; o3++) {
         for (k1 = 0; k1 < 5; k1++) {
         for (k2 = 0; k2 < 5; k2++) {
            fun[5] = boole(o3, fun[k1], fun[k2]);
            found[fun[5]] = 1;
         }}}
      }}}
   }}}
   printf("  0 1 2 3 4 5 6 7 8 9 A B C D E F\n");
   for (i = 0; i < 16; i++) {
      printf("%X", i);
      for (j = 0; j < 16; j++)
         printf("%2d", found[16*i + j]);
      printf("\n");
   }
   return 0;
}
