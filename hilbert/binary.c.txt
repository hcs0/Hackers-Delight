/* Converts the unsigned integer k to binary character form.
Result is in string s of length len. */

void binary(unsigned int k, int len, char *s) {
   int i;

   s[len] = 0;
   for (i = len - 1; i >= 0; i--) {
      if (k & 1) s[i] = '1';
      else       s[i] = '0';
      k = k >> 1;
   }
}
