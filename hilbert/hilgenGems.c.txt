/* Not used in Hacker book. */
/* Recursively draws a Hilbert curve
     orient -- is either +1 or -1
     angle  -- is 0, 90, 180, or 270, direction of step
     level  -- is initially the desired recursion depth

step moves to the next pixel in the direction angle marks it
C.f Graphics Gems II p. 486
*/
#include <stdio.h>
#include <stdlib.h>

void binary(unsigned int k, int len, char *s);
void step(void);

int x = -1, y = 0, angle = 0;           /* Global variables. */
int i = 0;                              /* Counter of distance along curve. */
int blen;                               /* Length of bit strings to print. */

void hilbert(int orient, int level) {   /* orient = +1 ==> clockwise. */
   level = level - 1;
   if (level <= 0) return;
   angle += orient*90;
   hilbert(-orient, level);
   step();
   angle -= orient*90;
   hilbert(orient, level);
   step();
   hilbert(orient, level);
   angle -= orient*90;
   step();
   hilbert(-orient, level);
   angle += orient*90;
}
void step() {
   char ii[33], xx[17], yy[17];

   switch(angle % 360) {
      case   0:            x = x + 1; break;
      case  90: case -270: y = y + 1; break;
      case 180: case -180: x = x - 1; break;
      case 270: case  -90: y = y - 1; break;
      default: printf("Invalid angle = %d\n", angle); exit(1); break;
   }
   binary(i, 2*blen, ii);
   binary(x, blen, xx);
   binary(y, blen, yy);
   printf("%5d   %s   %s %s\n", angle, ii, xx, yy);
   i = i + 1;                           /* Increment distance along curve. */
}
int main(int argc, char *argv[]) {
   int level;

   level = atoi(argv[1]);
   blen = level - 1;
   step();                              /* Print initial point (0, 0). */
   hilbert(1, level);
   return 0;
}
