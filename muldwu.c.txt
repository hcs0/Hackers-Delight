// Computes the doubleword product of two words, unsigned.
// (32x32 ==> 64 multiplication).
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

// w (two words) gets the product of u and v (one word each).
// w[0] is the most significant word of the result, w[1] the least.
// (The words are in big-endian order).
// It is Knuth's Algorithm M from [Knu2] section 4.3.1.

void muldwu1(unsigned w[], unsigned u, unsigned v) {
   unsigned u0, u1, v0, v1, k, t;
   unsigned w1, w2, w3;

   u0 = u >> 16; u1 = u & 0xFFFF;
   v0 = v >> 16; v1 = v & 0xFFFF;

   t = u1*v1;
   w3 = t & 0xFFFF;             // (*)
   k = t >> 16;

   t = u0*v1 + k;
   w2 = t & 0xFFFF;
   w1 = t >> 16;

   t = u1*v0 + w2;
   k = t >> 16;

   w[0] = u0*v0 + w1 + k;
   w[1] = (t << 16) + w3;       // (*)
/* w[1] = u*v;                  // Alternative. */

   return;
}

int errors;

void check(unsigned *tab, unsigned *res) {
   if (tab[0] != res[0] || tab[1] != res[1]) {
      errors = errors + 1;
      printf("Error, should get %08x %08x,\n", tab[0], tab[1]);
      printf("              got %08x %08x.\n", res[0], res[1]);
   }
}

int main() {
   int i, n;
   static unsigned test[][4] = {
      {1, 1, 0,1},                      // 1*1 = 1
      {0, 0xFFFFFFFF, 0,0},             // 0*big = 0
      {7, 3, 0,21},                     // 7*3 = 21
      {0x5555, 0xAAAA, 0,0x38E31C72},
      {0x80000000, 0x80000000, 0x40000000,0},
      {0x12345678, 0x87654321, 0x9A0CD05,0x70B88D78},
      {0x89ABCDEF, 0xFEDCBA98, 0x890F2A50,0xAD05EBE8},
      {0x55555555, 0xAAAAAAAA, 0x38E38E38,0x71C71C72},
      {0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFE,0x00000001},
   };
   unsigned result[2];

   n = sizeof(test)/(4*4);      /* Number of test cases */
                                /* (without interchanging operands). */

   printf("muldwu1:\n");
   for (i = 0; i < n; i += 1) {
      muldwu1(result, test[i][0], test[i][1]);
      check(&test[i][2], result);
      muldwu1(result, test[i][1], test[i][0]);
      check(&test[i][2], result);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n);
}
