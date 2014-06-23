#!/usr/bin/python
""" Division by Multiplication Considered More Generally
   Handles arbitrary max dividends (not necessarily of the form
2**W - 1), and bignums.
   Computes the magic number for signed division, given a max value of
the magnitude of the dividend, and a divisor. The divisor must be > 0.
   This uses equations (5) and (6) on page 162, but with nc determined
from the maximum value of n, which is not necessarily 2**W -1 for W =
the word size of n. It finds the smallest number that can serve for a
multiplier for all n in the range -nmax <= n <= nmax.
   It is much simpler than the program given in Hacker's Delight
(Figure 10-1) because in Python, one need not be concerned about
overflowing the word size in which the computations are done.
  Check these:
  Example: magicg(255, 7) is a 9-bit number, but magicg(200, 7) is an
8-bit number.
   Also, magicg(127, 7) is an 8-bit number, but magicg(90, 7) is a
6-bit number.
   Good test case: magicg 61 55 = 0x4B, shift = 12,
This has the shift at the maximum value of 2*nbits = 2*6 (c.f. Hacker's
Delight 2nd ed. p. 213 eqn (8), with W = nbits + 1). """

import sys

# ----------------------------- magicg ---------------------------------

def magicg(nmax, d):
   nc = ((nmax + 1)//d)*d - 1
   nbits = len(bin(nmax)) - 2
   for p in range(0, 2*nbits + 1):
      if 2**p > nc*(d - (2**p)%d):
         m = (2**p + d - (2**p)%d)//d
         return (m, p)
   print "Can't find p, something is wrong."
   sys.exit(1)

# ------------------------------ main ----------------------------------

args = sys.argv[1:]             # This is to work with Python 2.6 and 2.7.
if len(args) == 1:
   args = args[0].split()

if len(args) != 2: print "Wrong number of args, need 2."; sys.exit(1)

nmax = eval(args[0])            # Max value of dividend.
d = eval(args[1])               # Divisor.
if (nmax <= 0 or d <= 0):
   print "Both arguments must be > 0."
   sys.exit(1)

(m, s) = magicg(nmax, d)
print "For nmax %d, divisor %d, magic number = hex %X (%d bits), shift = %d" % \
   (nmax, d, m, len(bin(m)) - 2, s)
sys.exit(0)
