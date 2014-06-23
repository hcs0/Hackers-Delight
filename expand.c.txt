/* Inverse of compress (right). */

#include <stdio.h>

/* Inverse of compress2 (HD page 151). This version has
no branches in the loop. Eight insns in the loop, giving
8*32 + 2 = 258 insns worst case. */

unsigned expand2(unsigned x, unsigned m) {
   unsigned r, s, b;    // Result, shift, mask bit.

   r = 0;
   s = 0;
   do {
      b = m & 1;
      r = r | ((x & b) << s);
      s = s + 1;
      x = x >> b;
      m = m >> 1;
   } while (m != 0);
   return r;
}

/* Inverse of compress4 (HD page 153).
   I don't know a good way to do this. Seems like you
have to move the bits that go a long distance first, to
avoid overwriting. Maybe that problem could be solved by
moving a bit by first subtracting it where it came from,
and then adding it into the target location, with both
subtraction and addition done with xor. Not sure. But we
also have the problem of figuring out which bits move an
odd number of positions, which an odd multiple of 2,
etc. I don't know how to do that.
   A solution from Joseph Allen first does part of the
compress right operation, to compute the five "move"
quantities mv, and saves them in an array. Then it
applies them to the data x in the reverse of the order in
which they were computed. Can one do better (mainly,
avoid saving lg(n) values)?
   Allen's solution did not work if the irrelevant bits
are nonzero, but below is a modification of his program
that does work in that case.
   The assignment to x in the second for-loop is doing
a MUX operation with mv selecting bits from either x or
x << (1 << i). I.e., letting y = x << (1 << i), it is
computing

               (x & ~mv) | (y & mv)

by means of the equivalent expression (which saves an
instruction on a machine that doesn't have "and with
complement")

                x ^ ((x ^ y) & mv). */


// ------------------------------ cut ----------------------------------
unsigned expand4(unsigned x, unsigned m) {
   unsigned m0, mk, mp, mv, t;
   unsigned array[5];
   int i;

   m0 = m;              // Save original mask.
   mk = ~m << 1;        // We will count 0's to right.

   for (i = 0; i < 5; i++) {
      mp = mk ^ (mk << 1);              // Parallel suffix.
      mp = mp ^ (mp << 2);
      mp = mp ^ (mp << 4);
      mp = mp ^ (mp << 8);
      mp = mp ^ (mp << 16);
      mv = mp & m;                      // Bits to move.
      array[i] = mv;
      m = (m ^ mv) | (mv >> (1 << i));  // Compress m.
      mk = mk & ~mp;
   }

   for (i = 4; i >= 0; i--) {
      mv = array[i];
      t = x << (1 << i);
      x = (x & ~mv) | (t & mv);
//    x = ((x ^ t) & mv) ^ x;           // Alternative for above line.
   }
   return x & m0;       // Clear out extraneous bits.
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
      0x00000001, 0x80000000, 0x80000000, // These first 4 cases will
      0x0000001F, 0x0010084A, 0x0010084A, // work with Allen's "scatter"
      0x0000FFFF, 0x55555555, 0x55555555, // function, because the
      0x00001FFF, 0x88E00F55, 0x88E00F55, // irrelevant HO bits are 0.
      0xFFFFFFFF, 0x80000000, 0x80000000,
      0xFFFFFFFF, 0x0010084A, 0x0010084A,
      0xFFFFFFFF, 0x55555555, 0x55555555,
      0xFFFFFFFF, 0x88E00F55, 0x88E00F55,
      0x01234567, 0x0000FFFF, 0x00004567,
      0x01234567, 0xFFFF0000, 0x45670000,
      0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFF,
      0,          0,          0,
      0,          0xFFFFFFFF, 0,
      0xFFFFFFFF, 0,          0,
      0x80000000, 0x80000000, 0,
      0x55555555, 0x55555555, 0x11111111,
      0x55555555, 0xAAAAAAAA, 0x22222222,
      0x789ABCDE, 0x0F0F0F0F, 0x0B0C0D0E,
      0x789ABCDE, 0xF0F0F0F0, 0xB0C0D0E0,
      0x92345678, 0x80000000, 0,
      0x12345678, 0xF0035555, 0x50021540,
      0x80000000, 0xF0035555, 0,
   };

   n = sizeof(test)/sizeof(test[0]);

   printf("expand2:\n");
   for (i = 0; i < n; i += 3) {
      r = expand2(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   printf("expand4:\n");
   for (i = 0; i < n; i += 3) {
      r = expand4(test[i], test[i+1]);
      if (r != test[i+2])
         error(test[i], test[i+1], r, test[i+2]);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n/3);
   return errors;
}
