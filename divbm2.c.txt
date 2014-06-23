/* Base -2 division, 32/32 ==> 32 bit.  Inputs n and d are
2's-complement integers and the result is a base-2 integer.
   Max line length is 57, to fit in hacker.book. */
#include <stdio.h>

int divbm2(int n, int d) {         // q = n/d in base -2.
   int r, dw, c, q, i;

   r = n;                          // Init. remainder.
   dw = (-128)*d;                  // Position d.
   c = (-43)*d;                    // Init. comparand.
   if (d > 0) c = c + d;
   q = 0;                          // Init. quotient.
   for (i = 7; i >= 0; i--) {
      if (d > 0 ^ (i&1) == 0 ^ r >= c) {
         q = q | (1 << i);         // Set a quotient bit.
         r = r - dw;               // Subtract d shifted.
      }
      dw = dw/(-2);                // Position d.
      if (d > 0) c = c - 2*d;      // Set comparand for
      else c = c + d;              // next iteration.
      c = c/(-2);
   }
   return q;                       // Return quotient in
                                   // base -2.
                                   // Remainder is r,
}                                  // 0 <= r < |d|.

int main() {
   static const int a[] =
      {0,1,0, 1,1,1, 1,-1,3, -1,1,3, -1,-1,1,
       0,3,0, 1,3,0, 2,3,0, 3,3,1, 4,3,1, 5,3,1, 6,3,6, 7,3,6, 8,3,6,
       9,3,7, -1,3,3, -2,3,3, -3,3,3, -4,3,2,
       0,-3,0, 1,-3,0, 2,-3,0,
       3,-3,3, 4,-3,3, -1,-3,1, -2,-3,1, -3,-3,1, -4,-3,6, -5,-3,6, -6,-3,6,
       21,4,5, 21,3,27, 21,-4,15, 21,-3,9,
       76,5,19, 77,5,19, 78,5,19, 79,5,19, 80,5,16, 84,5,16, 85,5,17};
   int n, i, result;

   n = sizeof(a)/sizeof(a[0]);          // Number items in array.

   for (i = 0; i < n; i += 3) {
      result = divbm2(a[i], a[i+1]);
      if (result != a[i+2]) goto error;
   }

   printf("Passed all tests, returning rc = 0.");
   return 0;
error:
   printf("Error occurred for case (%d)/(%d), got %d\n",
          a[i], a[i+1], result);
   return i;
}
