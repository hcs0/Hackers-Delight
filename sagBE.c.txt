/* Big-endian version of code from GLS, 10/12/02. */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

char * binary(unsigned k);      // These are below.
unsigned compress(unsigned x, unsigned m);
unsigned compress_left(unsigned x, unsigned m);

unsigned SAG(unsigned x, unsigned m) {
   return compress(x, m) | compress_left(x, ~m);
}

int main(void)
{
   unsigned x0, x;
   static unsigned p[5] = {0x55555555,  /* This is for rotate left, */
                           0x33333333,  /* BE version of SAG, */
                           0xF0F0F0F0,  /* BE bit numbering. */
                           0xF00FF00F,
                           0xF0000FFF};

   static unsigned q[5] = {0x55555555,  /* This is for rotate left, */
                           0x33333333,  /* BE version of SAG, */
                           0xF0F0F0F0,  /* BE bit numbering. */
                           0xF00FF00F,
                           0xF0000FFF};

   x0 = 0x01234567;

   x = SAG(x0, p[0]);
   p[1] = SAG(p[1], p[0]);
   p[2] = SAG(p[2], p[0]);
   p[3] = SAG(p[3], p[0]);
   p[4] = SAG(p[4], p[0]);

   x = SAG(x, p[1]);
   p[2] = SAG(p[2], p[1]);
   p[3] = SAG(p[3], p[1]);
   p[4] = SAG(p[4], p[1]);

   x = SAG(x, p[2]);
   p[3] = SAG(p[3], p[2]);
   p[4] = SAG(p[4], p[2]);

   x = SAG(x, p[3]);
   p[4] = SAG(p[4], p[3]);

   x = SAG(x, p[4]);

   printf("x0 = %s\n", binary(x0));
   printf("x  = %s\n", binary(x));

   if (x != 0x12345670) printf("Error\n");
   else                 printf("OK\n");

   // Now check out the version with constants moved out of a loop.

   q[1] = SAG(q[1], q[0]);
   q[2] = SAG(SAG(q[2], q[0]), q[1]);
   q[3] = SAG(SAG(SAG(q[3], q[0]), q[1]), q[2]);
   q[4] = SAG(SAG(SAG(SAG(q[4], q[0]), q[1]), q[2]), q[3]);

   x = SAG(x0, q[0]);
   x = SAG( x, q[1]);
   x = SAG( x, q[2]);
   x = SAG( x, q[3]);
   x = SAG( x, q[4]);

   printf("x0 = %s\n", binary(x0));
   printf("x  = %s\n", binary(x));

   if (x != 0x12345670) printf("Error\n");
   else                 printf("OK\n");

   return 0;
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

unsigned compress(unsigned x, unsigned m) {
   unsigned mk, mp, mv, t;
   int i;

   x = x & m;           // Clear irrelevant bits.
   mk = ~m << 1;        // We will count 0's to right.

   for (i = 0; i < 5; i++) {
      mp = mk ^ (mk << 1);              // Parallel suffix.
      mp = mp ^ (mp << 2);
      mp = mp ^ (mp << 4);
      mp = mp ^ (mp << 8);
      mp = mp ^ (mp << 16);
      mv = mp & m;                      // Bits to move.
      m = m ^ mv | (mv >> (1 << i));    // Compress m.
      t = x & mv;
      x = x ^ t | (t >> (1 << i));      // Compress x.
      mk = mk & ~mp;
   }
   return x;
}

unsigned compress_left(unsigned x, unsigned m) {
   unsigned mk, mp, mv, t;
   int i;

   x = x & m;           // Clear irrelevant bits.
   mk = ~m >> 1;        // We will count 0's to left.

   for (i = 0; i < 5; i++) {
      mp = mk ^ (mk >> 1);              // Parallel prefix.
      mp = mp ^ (mp >> 2);
      mp = mp ^ (mp >> 4);
      mp = mp ^ (mp >> 8);
      mp = mp ^ (mp >> 16);
      mv = mp & m;                      // Bits to move.
      m = m ^ mv | (mv << (1 << i));    // Compress m.
      t = x & mv;
      x = x ^ t | (t << (1 << i));      // Compress x.
      mk = mk & ~mp;
   }
   return x;
}
