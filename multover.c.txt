#include <stdio.h>
unsigned nlz(int);
#pragma mc_func nlz { "7c630034" }      /* PowerPC cntlzw r3,r3. */
#pragma reg_killed_by nlz               /* Set reg kill to "none." */

int main(int argc, char* argv[], char* envp[]) {
   // Enter control-c to quit this program.

   unsigned x, y, z, m, n, t;

again:
   printf("Enter two hex operands\n");
   scanf("%x %x", &x, &y);

// ------------------------------ cut ----------------------------------
   m = nlz(x);
   n = nlz(y);
   if (m + n <= 30) goto overflow;
   t = x*(y >> 1);
   if ((int)t < 0) goto overflow;
   z = t*2;
   if (y & 1) {
      z = z + x;
      if (z < x) goto overflow;
   }
   // z is the correct product of x and y.
// ------------------------------ cut ----------------------------------

   printf("%x x %x = %x\n", x, y, z);
   goto again;

overflow:
   printf("Overflows\n");
   goto again;
}
