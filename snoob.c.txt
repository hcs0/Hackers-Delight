/* Computes the next number with the same number of one-bits.  Or, the
next subset of the same size as the given one. */

#include <stdio.h>
#include <stdlib.h>

int ntz(unsigned x) {         // Number of trailing zeros.
   int n;

   if (x == 0) return(32);
   n = 1;
   if ((x & 0x0000FFFF) == 0) {n = n +16; x = x >>16;}
   if ((x & 0x000000FF) == 0) {n = n + 8; x = x >> 8;}
   if ((x & 0x0000000F) == 0) {n = n + 4; x = x >> 4;}
   if ((x & 0x00000003) == 0) {n = n + 2; x = x >> 2;}
   return n - (x & 1);
}

int nlz(unsigned x) {
   int n;

   if (x == 0) return(32);
   n = 0;
   if (x <= 0x0000FFFF) {n = n +16; x = x <<16;}
   if (x <= 0x00FFFFFF) {n = n + 8; x = x << 8;}
   if (x <= 0x0FFFFFFF) {n = n + 4; x = x << 4;}
   if (x <= 0x3FFFFFFF) {n = n + 2; x = x << 2;}
   if (x <= 0x7FFFFFFF) {n = n + 1;}
   return n;
}

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

/* This version is from Hakmem (item 175) converted to C.  Caution:  Do
not call this with x = 0; it then does a divide by 0.  If called with
the largest number with a certain number of 1's (i.e., 11...100...0),
it returns a number smaller than the argument.
   It is basically seven ops, one of which is division. */

// ------------------------------ cut ----------------------------------
unsigned snoob(unsigned x) {
   unsigned smallest, ripple, ones;
                                // x = xxx0 1111 0000
   smallest = x & -x;           //     0000 0001 0000
   ripple = x + smallest;       //     xxx1 0000 0000
   ones = x ^ ripple;           //     0001 1111 0000
   ones = (ones >> 2)/smallest; //     0000 0000 0111
   return ripple | ones;        //     xxx1 0000 0111
}
// ---------------------------- end cut --------------------------------

/* Variation 1: ntz to avoid division.  Eight ops. */

unsigned snoob1(unsigned x) {
   unsigned smallest, ripple, ones;
                                // x = xxx0 1111 0000
   smallest = x & -x;           //     0000 0001 0000
   ripple = x + smallest;       //     xxx1 0000 0000
   ones = x ^ ripple;           //     0001 1111 0000
   ones = ones >> (2 + ntz(x)); //     0000 0000 0111
   return ripple | ones;        //     xxx1 0000 0111
}

/* Variation 2: nlz to avoid division.  Nine ops. */

unsigned snoob2(unsigned x) {
   unsigned smallest, ripple, ones;
                                // x = xxx0 1111 0000
   smallest = x & -x;           //     0000 0001 0000
   ripple = x + smallest;       //     xxx1 0000 0000
   ones = x ^ ripple;           //     0001 1111 0000
   ones = ones>>(33-nlz(smallest)); // 0000 0000 0111
   return ripple | ones;        //     xxx1 0000 0111
}

/* Variation 3: pop to avoid division.  Nine ops. */

unsigned snoob3(unsigned x) {
   unsigned smallest, ripple, ones;
                                // x = xxx0 1111 0000
   smallest = x & -x;           //     0000 0001 0000
   ripple = x + smallest;       //     xxx1 0000 0000
   ones = x ^ ripple;           //     0001 1111 0000
   ones = (1 <<                 //     0000 0000 0111
          (pop(ones) - 2)) - 1;
   return ripple | ones;        //     xxx1 0000 0111
}

/* The version below is from Harbison & Steele Fourth Ed. section 7.6.7
(p. 215).  Nine ops, not counting the "if" statement. */

unsigned next_set_of_n_elements(unsigned x) {
   unsigned smallest, ripple, new_smallest, ones;

   if (x == 0) return 0;
   smallest     = (x & -x);
   ripple       = x + smallest;
   new_smallest = (ripple & -ripple);
   ones         = ((new_smallest/smallest) >> 1) - 1;
   return ripple | ones;
}

/* I ran into this next version in October 2007 on the TopCoder web site:
http://forums.topcoder.com/?module=Message&messageID=574258
   The author is David de Kloet. The variable x must be signed,
or after generating a correct sequence, it will generate some incorrect
values and then loop forever. Must have x != 0. After generating the
sequence, it generates 0xFFFFFFFF and sticks at that value.
   The number of shifts done in the while-loop is equal to the number of
trailing zeros in the input x. So what is the average? I don't know,
will work on it. The values of x are NOT random, they tend to have more
trailing 0's than purely random numbers would have. (For uniformly
distributed random numbers, the average is 1.)
   For a word size of 32, if n (the number of 1-bits) = 1, the average
number of trailing 0's is 15.5 (average of the numbers from 0 t 31). For
n = 2, the average is 10 (I think). It gets lower for higher values of
n. */

int snoob4 (int x) {
   int y = x + (x & -x);
   x = x & ~y;
   while ((x & 1) == 0) x  = x >> 1;
   x = x >> 1;
   return y | x;
}

int main(int argc, char *argv[]) {
   int n;
   unsigned x, y, z, u, v, w;

   if (argc != 2) {
      printf("Need exactly one argument, an integer from 1 to 7.\n");
      exit(1);
   }

   n = strtol(argv[1], NULL, 10);
   if (n < 1 || n > 7) {
      printf("Argument must be an integer from 1 to 7.\n");
      exit(1);
   }

   printf("n = %d\n", n);

   x = (1 << n) - 1;
   y = x;
   z = x;
   u = x;
   v = x;
   w = x;
   do {
      printf("x, y, z, u, v, w = %02X %02X %02X %02X %02X %02X\n", x, y, z, u, v, w);
      y = snoob1(x);
      z = snoob2(x);
      u = snoob3(x);
      v = next_set_of_n_elements(x);
      w = snoob4(x);
      x = snoob(x);
   } while (x <= 255);
   return 0;
}
