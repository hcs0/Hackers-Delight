#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
/*
Given the "order" n of a Hilbert curve and a distance s along the curve,
this program computes the corresponding (x, y) coordinates.  The square
that the Hilbert curve traverses is of size 2**n by 2**n.
   The method is to employ the following state transition table:

If the current   And the next two     then       and enter
  state is        bits of s are      output        state
----------------------------------------------------------
     A                  00             00            B
     A                  01             01            A
     A                  10             11            A
     A                  11             10            D
     B                  00             00            A
     B                  01             10            B
     B                  10             11            B
     B                  11             01            C
     C                  00             11            D
     C                  01             10            C
     C                  10             00            C
     C                  11             01            B
     D                  00             11            C
     D                  01             01            D
     D                  10             00            D
     D                  11             10            A

The states correspond to mappings, with state A denoting the map from
binary 00 to 00, 01 to 01, 10 to 11, and 11 to 10, and similarly for
states B, C, and D.
   To use the table, start in state A.  Scan the bits of s in pairs from
left to right.  The first row means that if the current state is A and
the currently scanned bits of s are 00, then output 00 and enter state
B.  Then, advance to the next two bits of s.  The third row means that
if the current state is A and the scanned bits are 10, then output 11
and stay in state A.
   If the outputs are accumulated in left-to-right order, then when the
end of s is reached, the output quantity will contain x in the odd
numbered bit positions, and y in the even numbered bit positions.  For
example, suppose

                              s = 110100.

Then since the process starts in state A and the initial bits scanned
are 11, the process outputs 10 and enters state D (fourth row).  Then,
being in state D and scanning 01, the process outputs 01 and stays in
state D.  Lastly, the process outputs 11 and enters state C, although
the state is now immaterial.
   Thus the output is 100111.  From the odd and even bits respectively,
this gives x = 101 and y = 011.  Thus the (x, y) coordinates for s = 52
(110100) are (5, 3). */

// ------------------------- hil_xy_from_s -----------------------------

// Num ops: TBD

// ------------------------------ cut ----------------------------------
void hil_xy_from_s(unsigned s, int n, unsigned *xp,
                                      unsigned *yp) {

   int i;
   unsigned state, x, y, row;

   state = 0;                            // Initialize.
   x = y = 0;

   for (i = 2*n - 2; i >= 0; i -= 2) {   // Do n times.
      row = 4*state | (s >> i) & 3;      // Row in table.
      x = (x << 1) | (0x936C >> row) & 1;
      y = (y << 1) | (0x39C6 >> row) & 1;
      state = (0x3E6B94C1 >> 2*row) & 3; // New state.
   }
   *xp = x;                              // Pass back
   *yp = y;                              // results.
}
// ------------------------------ cut ----------------------------------

// ------------------------------ main ---------------------------------

int main(int argc, char *argv[]) {
   unsigned n, N, s, x, y;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                        // N = 2**2n.
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   printf("    s     x     y\n");
   for (s = 0; s < N; s++) {
      hil_xy_from_s(s, n, &x, &y);
      printf("%5d %5d %5d\n", s, x, y);
   }
   return 0;
}
