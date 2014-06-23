#include <stdio.h>
/* Round down to a power of 2. */

unsigned flp2(unsigned x) {
   x = x | (x >> 1);
   x = x | (x >> 2);
   x = x | (x >> 4);
   x = x | (x >> 8);
   x = x | (x >>16);
   return x - (x >> 1);
}

int errors;
void main() {
   int i;
   static unsigned test[] = {0,0, 1,1, 2,2, 3,2, 4,4, 5,4, 7,4, 8,8,
      9,8, 15,8, 16,16, 0xffff,0x8000, 0x7fffffff,0x40000000,
      0x80000000,0x80000000, 0x80000001,0x80000000,
      0xffffffff,0x80000000};
   void error(int x);

   for (i = 0; i < sizeof(test)/4; i += 2) {
      if (flp2(test[i]) != test[i+1]) error(test[i]);
   }
   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
}

void error(int x) {
   errors = errors + 1;
   printf("Error for x = %d, got %d\n", x, flp2(x));
}
