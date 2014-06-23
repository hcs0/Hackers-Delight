/* This file contains several programs for computing the number of
1-bits in an array of fullwords. Most of these programs are variations
of the Harley/Seal method, which uses a carry-save adder. This is quite
superior to the method given in Hacker's Delight, first edition, pp 72-73.
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// ------------------------------ pop ----------------------------------

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x >> 8);
   x = x + (x >> 16);
   return x & 0x0000003F;
}
/* Note: an alternative to the last three executable lines above is:
   return x*0x01010101 >> 24;
if your machine has a fast multiplier (suggested by Jari Kirma). */

// --------------------------- popArray1 -------------------------------

/* This is the naive method, simply evaluating the population count for
each word of the array, and adding. Ignoring loop control and loads,
this is 1 + p ops per word of the array. For p = 15 (code of Fig. 5-2,
inlined and with loads of constants moved out of the loop) this is 16
ops per word. */

int popArray1(unsigned A[], int n) {
   int i, tot;

   tot = 0;
   for (i = 0; i < n; i++)
      tot = tot + pop(A[i]);
   return tot;
}

// --------------------------- popArray2 -------------------------------

/* This is Harley's basic method. It combines groups of three array
elements into two words to which pop(x) is applied. The running time,
ignoring loop control and loads, is 7 elementary ops plus 2 pop counts
for each 3 array elements, i.e., (7 + 2p)/3 ops per word, where p is the
number of ops for one population count. For p = 15 (code of Fig. 5-2,
inlined) this is 37/3 = 12.33 ops per word. */

#define CSA(h,l, a,b,c) \
   {unsigned u = a ^ b; unsigned v = c; \
      h = (a & b) | (u & v); l = u ^ v;}

int popArray2(unsigned A[], int n) {

   int tot1, tot2, i;
   unsigned ones, twos;

   tot1 = 0;
   tot2 = 0;
   for (i = 0; i <= n - 3; i = i + 3) {
      CSA(twos, ones, A[i], A[i+1], A[i+2])
      tot1 = tot1 + pop(ones);
      tot2 = tot2 + pop(twos);
   }
   for (i = i; i < n; i++)      // Add in the last
      tot1 = tot1 + pop(A[i]);  // 0, 1, or 2 elements.

   return 2*tot2 + tot1;
}

// --------------------------- popArray3 -------------------------------

/* This is Harley's basic method but used in a different way (at Seal's
suggestion) than that of the above function. It brings in values from
the array two at a time and combines them with "ones" and "twos". The
array is assumed to have at least one element.
   The running time, ignoring loop control and loads, is 6 elementary
ops plus 1 pop count for each 2 array elements, i.e., (6 + p)/2 ops per
word, where p is the number of ops for one population count. For p = 15
(code of Fig. 5-2, inlined) this is 21/2 = 10.5 ops per word. */

int popArray3(unsigned A[], int n) {

   int tot, i;
   unsigned ones, twos;

   tot = 0;                     // Initialize.
   ones = 0;
   for (i = 0; i <= n - 2; i = i + 2) {
      CSA(twos, ones, ones, A[i], A[i+1])
      tot = tot + pop(twos);
   }
   tot = 2*tot + pop(ones);

   if (n & 1)                   // If there's a last one,
      tot = tot + pop(A[i]);    // add it in.

   return tot;
}

// --------------------------- popArray4 -------------------------------

/* This is similar to the above but it brings in array elements 4 at a
time and combines them with "ones", "twos", and "fours". Harley gave
this algorithm. The array is assumed to have at least three elements.
   The running time, ignoring loop control and loads, is 16 elementary
ops plus 1 pop count for each 4 array elements, i.e., (16 + p)/4 ops per
word, where p is the number of ops for one population count. For p = 15
(code of Fig. 5-2, inlined) this is 31/4 = 7.75 ops per word. */

int popArray4(unsigned A[], int n) {

   int tot, i;
   unsigned ones, twos, twosA, twosB, fours;

   tot = 0;                     // Initialize.
   twos = ones = 0;
   for (i = 0; i <= n - 4; i = i + 4) {
      CSA(twosA, ones, ones, A[i], A[i+1])
      CSA(twosB, ones, ones, A[i+2], A[i+3])
      CSA(fours, twos, twos, twosA, twosB)
      tot = tot + pop(fours);
   }
   tot = 4*tot + 2*pop(twos) + pop(ones);

   for (i = i; i < n; i++)      // Simply add in the last
      tot = tot + pop(A[i]);    // 0, 1, 2, or 3 elements.
   return tot;
}

// --------------------------- popArray5 -------------------------------

/* At the risk of being a bore, the function below is similar to that
above, but it brings in array elements 8 at a time and combines them
with "ones", "twos", "fours", and "eights". The array is assumed to have
at least seven elements.
   The running time, ignoring loop control and loads, is 36 elementary
ops plus 1 pop count for each 8 array elements, i.e., (36 + p)/8 ops per
word, where p is the number of ops for one population count. For p = 15
(code of Fig. 5-2, inlined) this is 51/8 = 6.375 ops per word. */

int popArray5(unsigned A[], int n) {

   int tot, i;
   unsigned ones, twos, twosA, twosB,
      fours, foursA, foursB, eights;

   tot = 0;                     // Initialize.
   fours = twos = ones = 0;

   for (i = 0; i <= n - 8; i = i + 8) {
      CSA(twosA, ones, ones, A[i], A[i+1])
      CSA(twosB, ones, ones, A[i+2], A[i+3])
      CSA(foursA, twos, twos, twosA, twosB)
      CSA(twosA, ones, ones, A[i+4], A[i+5])
      CSA(twosB, ones, ones, A[i+6], A[i+7])
      CSA(foursB, twos, twos, twosA, twosB)
      CSA(eights, fours, fours, foursA, foursB)
      tot = tot + pop(eights);
   }
   tot = 8*tot + 4*pop(fours) + 2*pop(twos) + pop(ones);

   for (i = i; i < n; i++)      // Simply add in the last
      tot = tot + pop(A[i]);    // 0 to 7 elements.
   return tot;
}

// --------------------------- popArray6 -------------------------------

/* This function generalizes the pattern illustrated by the function
above, with the result that the bits in an n-word array can be counted
with ceil(log2(n+3)) evaluations of population count.
   The inner loop (with the CSA) is done very close to 2 times for each
outer loop iteration. This is based on both a mathematical calculation
(which shows something less than 9/4 times) and instrumentation (e.g.,
for n = 10,000 the inner loop is executed 9983 times). The inner loop
compiles into 19 instructions, mostly housekeeping (shifts, adds loads,
stores). This results in a time of 2*19 + 19 = 57 instructions for each
outer loop iteration, or 28.5 instructions per word of the array.
This is worse than the naive method, and MUCH worse than the above
program, which compiles into 8.0 instructions/word. So this routine is
a bad idea unless the time to do a population count on one word is very
large (greater than 30 instructions anyway).
   Haven't timed the other loop, but it is executed only about log2(n)
times and so isn't so important.
   Therefore, if this method is to be useful, the housekeeping steps
must be greatly reduced. It may be possible to do this by "unwinding"
the first few inner loop iterations. Not sure how good the result would
be. */

int popArray6(unsigned A[], int n) {

   int tot, i, k;
   unsigned z, hi, lo;
   char nrow[30]; unsigned sum[30][2];

   memset(nrow, 0, sizeof(nrow));       // Clear "nrow".

   sum[0][0] = 0;               // Init. by putting in
   nrow[0] = 1;                 // a fake 0-element.

   for (i = 0; i <= n-2; i = i + 2){
      sum[0][1] = A[i];
      z = A[i+1];
      k = 0;
      do {
         CSA(z, sum[k][0], sum[k][0], sum[k][1], z)
         nrow[k] = 1;
         k = k + 1;
      } while (nrow[k] == 2);
      sum[k][nrow[k]] = z;
      nrow[k] = nrow[k] + 1;
   }

   if (i == n - 1) {            // If there's one more in
      sum[0][1] = A[i];         // the array, put it in
      nrow[0] = 2;              // sum[0][1].
   }

   /* Make a pass over the "sum" array compressing all
   rows that have two entries to the same row but having
   only one entry, while adding an entry to the
   subsequent row. This can make the subsequent row have,
   in effect, three entries, which we similarly compress.
   Compute the total during this pass. When an empty row
   is encountered, we're done. */

   tot = 0;
   hi = 0;
   for (k = 0; nrow[k] != 0; k++) {
      if (nrow[k] == 1) z = 0;
      else              z = sum[k][1];  // (Is 2.)

      CSA(hi, lo, sum[k][0], z, hi)
      tot = tot + (pop(lo) << k);
   }

   tot = tot + (pop(hi) << k);

   return tot;
}

// ------------------------------ main ---------------------------------

int main(void) {
   unsigned A[101];
   int i, n, s1, s;

   n = sizeof(A)/sizeof(A[0]);
   A[0] = 0xFFFFFFFF;                   // Fill the array
   A[1] = 5;                            // with somewhat
// printf("%08x\n", A[0]);              // random numbers.
// printf("%08x\n", A[1]);
   for (i = 2; i < n; i++) {
      A[i] = rand();
//    printf("%08x\n", A[i]);
   }
   s1 = popArray1(A, n);
   printf("Array size = %d, pop count = %d\n", n, s1);

   s = popArray2(A, n);
   if (s == s1) printf("popArray2 is ok.\n");
   else         printf("popArray2 = %d, ERROR.\n", s);

   s = popArray3(A, n);
   if (s == s1) printf("popArray3 is ok.\n");
   else         printf("popArray3 = %d, ERROR.\n", s);

   s = popArray4(A, n);
   if (s == s1) printf("popArray4 is ok.\n");
   else         printf("popArray4 = %d, ERROR.\n", s);

   s = popArray5(A, n);
   if (s == s1) printf("popArray5 is ok.\n");
   else         printf("popArray5 = %d, ERROR.\n", s);

   s = popArray6(A, n);
   if (s == s1) printf("popArray6 is ok.\n");
   else         printf("popArray6 = %d, ERROR.\n", s);

   return s - s1;
}
