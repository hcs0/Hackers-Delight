/* From GLS, 10/12/02. */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

char * binary(unsigned k);              // Is below.

/* Code from GLS.  Nine insns in the loop, giving 9*32 + 3 = 291 insns
worst case (mask = all 1's, not counting subroutine linkage). */

unsigned compress1(unsigned x, unsigned mask) {
  unsigned result = 0, bit = 1;
  while (mask != 0) {
    if ((mask & 1) != 0) {
      if (x & 1) result |= bit;
      bit <<= 1;
    }
    mask >>= 1;
    x >>= 1;
  }
  return result;
}

/* A version with no branches in the loop.  Eight insns in the loop,
giving 8*32 + 2 = 258 insns worst case (however, the code above might
be faster if the mask is sparse). */

// ------------------------------ cut ----------------------------------
unsigned compress2(unsigned x, unsigned m) {
   unsigned r, s, b;    // Result, shift, mask bit.

   r = 0;
   s = 0;
   do {
      b = m & 1;
      r = r | ((x & b) << s);
      s = s + b;
      x = x >> 1;
      m = m >> 1;
   } while (m != 0);
   return r;
}
// ---------------------------- end cut --------------------------------

/* Code from GLS.  Runs on a basic RISC in 159 ops total, incl.
subroutine overhead (just 3 ops).  Makes it clear that the five
masks can be precomputed if the mask is known (the five masks are
independent of x).  But this costs 5 stores and 5 loads because
"masks" is an array.  */

unsigned compress3(unsigned x, unsigned mask) {
  unsigned masks[5];
  unsigned q, m, zm;
  int i;
  m = ~mask;
  zm = mask;
  for (i = 0; i < 5; i++) {
      q = m;
      m ^= m << 1;
      m ^= m << 2;
      m ^= m << 4;
      m ^= m << 8;
      m ^= m << 16;
      masks[i] = (m << 1) & zm;
      m = q & ~m;
      q = zm & masks[i]; zm = zm ^ q ^ (q >> (1 << i));
  }
  x = x & mask;
  q = x & masks[0];  x = x ^ q ^ (q >> 1);
  q = x & masks[1];  x = x ^ q ^ (q >> 2);
  q = x & masks[2];  x = x ^ q ^ (q >> 4);
  q = x & masks[3];  x = x ^ q ^ (q >> 8);
  q = x & masks[4];  x = x ^ q ^ (q >> 16);
  return x;
}

/* Modification of GLS's code in which last 5 lines are
merged into the loop, to avoid the stores and loads of
array "masks."  Num. insns. = 5*24 + 7 = 127 total
(compiled to Cyclops and adding 1 for the "andc" op).
   Michael Dalton has observed that the first shift left
can be omitted. */

// ------------------------------ cut ----------------------------------
unsigned compress4(unsigned x, unsigned m) {
   unsigned mk, mp, mv, t;
   int i;

   x = x & m;           // Clear irrelevant bits.
   mk = ~m << 1;        // We will count 0's to right.

   printf("\n\n           m = %s\n", binary(m));
   printf("           x = %s\n", binary(x));
   for (i = 0; i < 5; i++) {
      printf("\ni = %d,    mk = %s\n", i, binary(mk));
      mp = mk ^ (mk << 1);              // Parallel suffix.
      mp = mp ^ (mp << 2);
      mp = mp ^ (mp << 4);
      mp = mp ^ (mp << 8);
      mp = mp ^ (mp << 16);
      printf("After PP, mp = %s\n", binary(mp));
      mv = mp & m;                      // Bits to move.
      printf("          mv = %s\n", binary(mv));
      m = m ^ mv | (mv >> (1 << i));    // Compress m.
      t = x & mv;
      x = x ^ t | (t >> (1 << i));      // Compress x.
      printf("           m = %s\n", binary(m));
      printf("           x = %s\n", binary(x));
      mk = mk & ~mp;
   }
   return x;
}
// ---------------------------- end cut --------------------------------

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
//       Data        Mask       Result
      0xFFFFFFFF, 0x80000000, 0x00000001,
      0xFFFFFFFF, 0x0010084A, 0x0000001F,
      0xFFFFFFFF, 0x55555555, 0x0000FFFF,
      0xFFFFFFFF, 0x88E00F55, 0x00001FFF,
      0x01234567, 0x0000FFFF, 0x00004567,
      0x01234567, 0xFFFF0000, 0x00000123,
      0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFF,
      0,          0,          0,
      0,          0xFFFFFFFF, 0,
      0xFFFFFFFF, 0,          0,
      0x80000000, 0x80000000, 1,
      0x55555555, 0x55555555, 0x0000FFFF,
      0x55555555, 0xAAAAAAAA, 0,
      0x789ABCDE, 0x0F0F0F0F, 0x00008ACE,
      0x789ABCDE, 0xF0F0F0F0, 0x000079BD,
      0x92345678, 0x80000000, 0x00000001,
      0x12345678, 0xF0035555, 0x000004ec,
      0x80000000, 0xF0035555, 0x00002000,
   };

   n = sizeof(test)/sizeof(test[0]);

   printf("compress1:\n");
   for (i = 0; i < n; i += 3) {
      r = compress1(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   printf("compress2:\n");
   for (i = 0; i < n; i += 3) {
      r = compress2(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   printf("compress3:\n");
   for (i = 0; i < n; i += 3) {
      r = compress3(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   printf("compress4:\n");
   for (i = 0; i < n; i += 3) {
      r = compress4(test[i], test[i+1]);
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
