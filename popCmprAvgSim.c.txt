/* Given a word size n, this program computes, by simulation, the
average minimal population count of two random n-bit words each of which
has been AND'ed with the complement of the other so that the words are
never both 1 in the same bit position. This is necessary to estimate the
running time (number of loop iterations) of an algorithm that determines
which of two words has the lower population count.
   The true averages, obtained by running an exhaustive test, are:

   n    true average
   2    0.125
   3    0.28125
   4    0.453125
   5    0.634766
   6    0.823242
   7    1.016846
   8    1.214478
   9    1.415382
  10    1.619015
  11    1.824965
  12    2.032918
  13    2.242623
  14    2.453878
  15    2.666517
  16    2.880401
  17    3.095413 12 min on Thinkpad model T30
  24    4.625    Simulated, 100,000,000 trials
  32    6.186    Simulated, 100,000,000 trials
*/

#include <stdio.h>
#include <stdlib.h>     //To define "strtol".

int pop(unsigned x) {           // Alg. of Fig. 5-2. */
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x >> 8);
   x = x + (x >> 16);
   return x & 0x0000003F;
}

int main(int argc, char *argv[])
{
   char **p, *q;
   int n, i, nbx, nby, nb, ntrials;
   unsigned mask, xp, yp, x, y;
   unsigned long long tot;
   double avg;

   if (argc != 3) {
      fprintf(stderr, "Need exactly two arguments, num bits and num trials.");
      return 1;
   }

   p = &q;
   n = strtol(argv[1], p, 0);
   if (q == argv[1]) {
      fprintf(stderr, "Invalid first argument.");
      return 1;
   }

   if (n <= 0 || n > 32) {
      fprintf(stderr, "First arg not in range 1 to 32.");
      return 1;
   }

   ntrials = strtol(argv[2], p, 0);
   if (q == argv[2]) {
      fprintf(stderr, "Invalid second argument %d.", ntrials);
      return 1;
   }

   if (ntrials <= 0 || ntrials > 100000000) {
      fprintf(stderr, "Second arg not in range 1 to 100,000,000.");
      return 1;
   }

   mask = 2*(1 << (n-1)) - 1;
   printf("n = %d, mask = %08x, ntrials = %d\n", n, mask, ntrials);
   tot = 0;
   for (i = 0; i < ntrials; i++) {
      xp = rand() & mask;
      yp = rand() & mask;
      x = xp & ~yp;
      y = yp & ~xp;
      nbx = pop(x);
      nby = pop(y);
      nb = nbx < nby ? nbx : nby;
      tot = tot + nb;
/*    printf("xp = %08x yp = %08x x = %08x y = %08x nb = %d tot = %d\n", */
/*    xp, yp, x, y, nb, tot); */
   }
   avg = (double)tot/ntrials;
   printf("avg = %f\n", avg);
}
