#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
/*
Given the "order" n of a Hilbert curve and (x, y) coordinates, this
program computes the distance s along the curve to the point (x, y).
The square that the Hilbert curve traverses is of size 2**n by 2**n.
   The method is to employ the following state transition table:

If the current   And the next bits   then append  and enter
  state is        of x and y are        to s        state
----------------------------------------------------------
     A                 (0, 0)            00           B
     A                 (0, 1)            01           A
     A                 (1, 0)            11           D
     A                 (1, 1)            10           A
     B                 (0, 0)            00           A
     B                 (0, 1)            11           C
     B                 (1, 0)            01           B
     B                 (1, 1)            10           B
     C                 (0, 0)            10           D
     C                 (0, 1)            11           B
     C                 (1, 0)            01           C
     C                 (1, 1)            00           D
     D                 (0, 0)            10           C
     D                 (0, 1)            01           D
     D                 (1, 0)            11           A
     D                 (1, 1)            00           C

The states correspond to mappings, with state A denoting the map from
binary 00 to 00, 01 to 01, 10 to 11, and 11 to 10, and similarly for
states B, C, and D.
   To use the table, start in state A.  Scan the bits of s in pairs from
left to right.  The first row means that if the current state is A and
the currently scanned bits of (x, y) are (0, 0), then output 00 and
enter state B. Then, advance to the next bits of (x, y).  The third row
means that if the current state is A and the scanned bits are (1, 0),
then output 11 and stay in state D.
   If the output is accumulated in left-to-right order, then when the
end of x and y is reached, the output quantity will contain the length s
of the curve from its beginning to (x, y).
For example, suppose the order is 3 and

                          (x, y) = (4, 3).

Then since the process starts in state A and the initial bits scanned
are (1, 0), the process outputs 11 and enters state D (third row).  Then,
being in state D and scanning (0, 1), the process outputs 01 and stays in
state D.  Lastly, the process outputs 01 and enters state D, although
the state is now immaterial.
   Thus the output is 110101, i.e., decimal 53. */

// ------------------------- hil_s_from_xy -----------------------------

// Num ops: TBD

// ------------------------------ cut ----------------------------------
unsigned hil_s_from_xy(unsigned x, unsigned y, int n) {

   int i;
   unsigned state, s, row;

   state = 0;                            // Initialize.
   s = 0;

   for (i = n - 1; i >= 0; i--) {
      row = 4*state | 2*((x >> i) & 1) | (y >> i) & 1;
      s = (s << 2) | (0x361E9CB4 >> 2*row) & 3;
      state = (0x8FE65831 >> 2*row) & 3;
   }
   return s;
}
// ------------------------------ cut ----------------------------------

// ------------------------------ main ---------------------------------

int main(int argc, char *argv[]) {
   unsigned n, N, x, y, s;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << n;                          // N = 2**n.
   printf("Distance along the Hilbert curve of order %d\n", n);
   printf("    x     y     s\n");
   for (x = 0; x < N; x++) {
      for (y = 0; y < N; y++) {
         s = hil_s_from_xy(x, y, n);
         printf("%5d %5d %5d\n", x, y, s);
      }
   }
   return 0;
}
