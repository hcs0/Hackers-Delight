#include <stdio.h>
#include <stdlib.h>
#include <time.h>

char * binary(unsigned k);              // Is below.

/* Michael Dalton has observed that the first shift right
can be omitted. */

unsigned compress_left(unsigned x, unsigned m) {
   unsigned mk, mp, mv, t;
   int i;

   x = x & m;           // Clear irrelevant bits.
   mk = ~m >> 1;        // We will count 0's to left.

   printf("\n\n           m = %s\n", binary(m));
   printf("           x = %s\n", binary(x));
   for (i = 0; i < 5; i++) {
      printf("\ni = %d,    mk = %s\n", i, binary(mk));
      mp = mk ^ (mk >> 1);              // Parallel prefix.
      mp = mp ^ (mp >> 2);
      mp = mp ^ (mp >> 4);
      mp = mp ^ (mp >> 8);
      mp = mp ^ (mp >> 16);
      printf("After PP, mp = %s\n", binary(mp));
      mv = mp & m;                      // Bits to move.
      printf("          mv = %s\n", binary(mv));
      m = m ^ mv | (mv << (1 << i));    // Compress m.
      t = x & mv;
      x = x ^ t | (t << (1 << i));      // Compress x.
      printf("           m = %s\n", binary(m));
      printf("           x = %s\n", binary(x));
      mk = mk & ~mp;
   }
   return x;
}

int errors;
void error(unsigned x, unsigned m, unsigned got, unsigned shdbe) {
   errors = errors + 1;
   printf("Error for x = %08X, m = %08x, got %08X, should be %08X\n",
          x, m, got, shdbe);
}

int main(void)
{
   int i, n;
   unsigned r;
   static unsigned test[] = {
      0xFFFFFFFF, 0x80000000, 0x80000000,
      0xFFFFFFFF, 0x0010084A, 0xF8000000,
      0xFFFFFFFF, 0x55555555, 0xFFFF0000,
      0xFFFFFFFF, 0x88E00F55, 0xFFF80000,
      0x01234567, 0x0000FFFF, 0x45670000,
      0x01234567, 0xFFFF0000, 0x01230000,
      0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFF,
      0,          0,          0,
      0,          0xFFFFFFFF, 0,
      0xFFFFFFFF, 0,          0,
      0x80000000, 0x80000000, 0x80000000,
      0x55555555, 0x55555555, 0xFFFF0000,
      0x55555555, 0xAAAAAAAA, 0,
      0x789ABCDE, 0x0F0F0F0F, 0x8ACE0000,
      0x789ABCDE, 0xF0F0F0F0, 0x79BD0000,
      0x92345678, 0x80000000, 0x80000000,
      0x12345678, 0xF0035555, 0x13B00000, /* Was 000004ec */
      0x80000000, 0xF0035555, 0x80000000,
   };

   n = sizeof(test)/sizeof(test[0]);

   printf("compress_left:\n");
   for (i = 0; i < n; i += 3) {
      r = compress_left(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n/3);
   return errors;
}

/* Converts the unsigned integer k to binary character form with a blank
after every fourth digit.  Result is in string s of length 39.  Caution:
If you want to save the string, you must move it.  This is intended for
use with printf, and you can have only one reference to this in each
printf statement. */

char * binary(unsigned k) {
   int i, j;
   static char s[40] = "0000 0000 0000 0000 0000 0000 0000 0000";

   j = 38;
   for (i = 31; i >= 0; i--) {
      if (k & 1) s[j] = '1';
      else       s[j] = '0';
      j = j - 1;
      k = k >> 1;
      if ((i & 3) == 0) j = j - 1;
   }
   return s;
}
