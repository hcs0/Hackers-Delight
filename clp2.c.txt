#include <stdio.h>
/* Round up to a power of 2. */

unsigned clp2(unsigned x) {
   x = x - 1;
   x = x | (x >> 1);
   x = x | (x >> 2);
   x = x | (x >> 4);
   x = x | (x >> 8);
   x = x | (x >>16);
   return x + 1;
}

int errors;
void main() {
   int i;
   static unsigned test[] = {0,0, 1,1, 2,2, 3,4, 4,4, 5,8, 7,8, 8,8,
      9,16, 15,16, 16,16, 0xffff,0x10000, 0x7fffffff,0x80000000,
      0x80000000,0x80000000, 0x80000001,0,
      0xffffffff,0};
   void error(int x);

   for (i = 0; i < sizeof(test)/4; i += 2) {
      if (clp2(test[i]) != test[i+1]) error(test[i]);
   }
   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
}

void error(int x) {
   errors = errors + 1;
   printf("Error for x = %d, got %d\n", x, clp2(x));
}
