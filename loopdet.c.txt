#include <stdio.h>
#include <stdlib.h>
#define max(a, b) ((a) >= (b) ? (a) : (b))

// Loop detectors of Floyd and Gosper.

int X0_in, X1_in, X2_in;

// Function f that generates the test sequence.

int f(int x) {
   if (x == X2_in) return X1_in;
   else return x + 1;
}

int nlz(unsigned x) {         // Number of leading zeros.
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

/* The loop detector below is given in Knuth Vol. 2, third
edition, section 3.1 problem 6, where it is attributed to
R. W. Floyd.  */

void
ld_Floyd(int (*f)(int), int X0, int *mu, int *lambda) {
   int Xi, X2i, Xn, Xnpi, Xmu, i, n;

   Xi = X0;
   X2i = X0;

   // Search for a repeating element, i.e., one for which
   // X[n] = X[2n] with n >= 1.

   for (i = 1; ; i++) {
      Xi = (*f)(Xi);
      X2i = (*f)((*f)(X2i));
      if (Xi == X2i) break;
   }
   n = i;                       // Set the position and
   Xn = Xi;                     // value of X[n], the re-
                                // peating element found.

   // Now find the first element of the repeating part.
   // by finding first place where X[i] = X[n+i].

   Xnpi = Xn;
   Xi = X0;
   for (i = 0; ; i++) {
      if (Xi == Xnpi) break;
      Xi = (*f)(Xi);
      Xnpi = (*f)(Xnpi);
   }
   *mu = i;                     // Give mu to caller.

   // Now find the period lambda by simply searching from
   // mu on to see where it first repeats.  (This is not
   // the method suggested by Knuth.)

   Xmu = Xi;
   for (i = 1; ; i++) {
      Xi = f(Xi);               // Really X[i+mu].
      if (Xi == Xmu) break;
   }
   *lambda = i;                 // Give lambda to caller.
}

/* The loop detector below is given in Hakmem item 132 by
R. W. Gosper, and in Knuth Vol. 2, third edition, answers
to exercises for section 3.1 problem 7 */

// ------------------------------ cut ----------------------------------
void ld_Gosper(int (*f)(int), int X0, int *mu_l,
                              int *mu_u, int *lambda) {
   int Xn, k, m, kmax, n, lgl;
   int T[33];

   T[0] = X0;
   Xn = X0;
   for (n = 1; ; n++) {
      Xn = f(Xn);
      kmax = 31 - nlz(n);           // Floor(log2 n).
      for (k = 0; k <= kmax; k++) {
         if (Xn == T[k]) goto L;
      }
      T[ntz(n+1)] = Xn;             // No match.
   }
L:
   // Compute m = max{i | i < n and ntz(i+1) = k}.

   m = ((((n >> k) - 1) | 1) << k) - 1;
   *lambda = n - m;
   lgl = 31 - nlz(*lambda - 1); // Ceil(log2 lambda) - 1.
   *mu_u = m;                       // Upper bound on mu.
   *mu_l = m - max(1, 1 << lgl) + 1;// Lower bound on mu.
}
// ------------------------------ cut ----------------------------------

/* User enters the starting number of a sequence, the
first value (not index) at which the repeated subsequence
begins, and the last value of the repeated subsequence.
*/

int main(int argc, char *argv[]) {
   int i, x;
   int mu, mu_l, mu_u, lambda;      // Results of search.

   if (argc <= 3) {printf("Need three parameters"); exit(1);}
   X0_in = strtol(argv[1], NULL, 10);
   X1_in = strtol(argv[2], NULL, 10);
   X2_in = strtol(argv[3], NULL, 10);
   if (X2_in < X1_in) {
      printf("Third parameter must be >= second parameter.\n");
      exit(1);
   }

   // First print out a few values of the sequence.
   x = X0_in;                   // Starting value X0.
   for (i = 0; i < 18; i++) {
      printf("i = %d, x = %d\n", i, x);
      x = f(x);
   }

   // Find the first repeated element and the period.

   ld_Floyd(&f, X0_in, &mu, &lambda);
   printf("Floyd,  lambda = %d, mu = %d\n", lambda, mu);
   ld_Gosper(&f, X0_in, &mu_l, &mu_u, &lambda);
   printf("Gosper, lambda = %d, mu_l = %d, mu_u = %d, spread = %d\n", lambda, mu_l, mu_u, mu_u-mu_l+1);
   return 0;
}
