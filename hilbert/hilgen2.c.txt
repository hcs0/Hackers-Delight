/* Compile with g++, and combine with hilgen1.cc. */
/* Mod of Graphics Gems in which angle is encoded as 0 - 3 rather than 0, 90, etc. */
/* Also, input arg is order. */
/* Recursively draws a Hilbert curve
     dir   -- is 0, 1, 2, 3 for right, up, left, down resp.
     rot   -- +1 for clockwise and -1 for counterclockwise
     order -- the order of the Hilbert curve to draw

step(dir) moves one unit in the direction "dir."
C.f Graphics Gems II p. 486, with minor modifications.
*/
#include <stdio.h>
#include <stdlib.h>

int x = -1, y = 0;              // Global variables.
int s = 0;                      // Dist. along curve.
int blen;                       // Length to print.

void hilbert(int dir, int rot, int order);

void binary(unsigned k, int len, char *s) {
/* Converts the unsigned integer k to binary character
form.  Result is string s of length len. */
   int i;

   s[len] = 0;
   for (i = len - 1; i >= 0; i--) {
      if (k & 1) s[i] = '1';
      else       s[i] = '0';
      k = k >> 1;
   }
}
void step(int dir) {
   char ii[33], xx[17], yy[17];

   switch(dir & 3) {
      case 0: x = x + 1; break;
      case 1: y = y + 1; break;
      case 2: x = x - 1; break;
      case 3: y = y - 1; break;
   }
   binary(s, 2*blen, ii);
   binary(x, blen, xx);
   binary(y, blen, yy);
   printf("%5d   %s   %s %s\n", dir, ii, xx, yy);
   s = s + 1;                   // Increment distance.
}
int main(int argc, char *argv[]) {
   int order;

   order = atoi(argv[1]);
   blen = order;
   step(0);                     // Print init. point.
   hilbert(0, 1, order);
   return 0;
}
