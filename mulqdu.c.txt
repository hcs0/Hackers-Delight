// Computes the quadword product of two doublewords, unsigned.
// (64x64 ==> 128 multiplication).
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

// W (four words) gets the product of U and V (two words each).
// W[0], U[0], and V[0] contain the MOST significant words.
// (The words are in big-endian order).
// This version works only on big-endian machines.
// It is Knuth's Algorithm M from [Knu2] section 4.3.1.

void mulqdu1(unsigned W[], unsigned U[], unsigned V[]) {
   unsigned short *w, *u, *v;
   unsigned int k, t;
   int i, j;

   w = (unsigned short *)W;     // w, u, v reference the
   u = (unsigned short *)U;     // fullword arrays as
   v = (unsigned short *)V;     // halfwords.

   W[2] = 0;
   W[3] = 0;

   for (j = 3; j >= 0; j--) {
      k = 0;
      for (i = 3; i >= 0; i--) {
         t = u[i]*v[j] + w[i + j + 1] + k;
         w[i + j + 1] = t & 0xFFFF;
         k = t >> 16;
      }
      w[j] = k;
   }
   return;
}

int errors;

void check(unsigned *tab, unsigned *res) {
   if (tab[0] != res[0] || tab[1] != res[1] ||
       tab[2] != res[2] || tab[3] != res[3]) {
      errors = errors + 1;
      printf("Error, should get %08x %08x %08x %08x,\n",
         tab[0], tab[1], tab[2], tab[3]);
      printf("              got %08x %08x %08x %08x.\n",
         res[0], res[1], res[2], res[3]);
   }
}

int main() {
   int i, n;
   static unsigned test[][8] = {
      {0,1, 0,1, 0,0,0,1},                    /* 1*1 = 1 */
      {0,0, 0xFFFFFFFF,0xFFFFFFFF, 0,0,0,0},  /* 0*big = 0 */
      {0,7, 0,3, 0,0,0,21},                   /* 7*3 = 21 */
      {0xFFFFFFFF,0xFFFFFFFF, 0xFFFFFFFF,0xFFFFFFFF, 0xFFFFFFFF,0xFFFFFFFE,0x00000000,0x00000001},
   };
   unsigned result[4];

   n = sizeof(test)/(8*4);      /* Number of test cases */
                                /* (without interchanging operands). */

   printf("mulqdu1:\n");
   for (i = 0; i < n; i += 1) {
      mulqdu1(result, &test[i][0], &test[i][2]);
      check(&test[i][4], result);
      mulqdu1(result, &test[i][2], &test[i][0]);
      check(&test[i][4], result);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n);
}
