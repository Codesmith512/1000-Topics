# IEEE 754 Floating Point Numbers

IEEE-754 defines a floating point format commonly found in computer science. It is also the primary format used in the x86 platform.

There are 4 types of numbers defined in the standard -- Normalized, Denormalized, Infinity, and NaN (Not a Number).

**Normalized**
```
1b    Sign        If set, number is negative
nb    Exponent    Determines the power of 2 the number is raised to
                  value-((2^n-1)-1)
                  ie 0x80 = 2^1
mb    Mantissa    Determines the fraction to be raised to 2
                  (value/(2^m))+1
```

This is the format almost all floats are utilized with. It provides some convienent optimizations at a low-level (not covered here) [1].

*A really convienent/useful property is that the mantissa is incremented as the float increases. When the mantissa overflows (would be 2.0), it is reset to 1.0, and the exponent is increased. (Which yields the exact same result, but reduces the precision of the mantissa from this point forward).*

**Denormalized**  
A number is denormalized if the exponent is 0.  
The number is encoded exactly as normal, except that the exponent value is assumed to be 1 instead of 0, and the mantissa encoding starts with 0.f

*Try to avoid these, as they are typically inefficient. This is because they require an entirely different set of equations to use because the representation is not equivalent to standard normalized numbers. Because these numbers are rarely used in consumer computing, most consumer/gaming CPUs/GPUs have extremely inefficient circuits for these to make more room to optimize the normalized numbers that are typically used.*

**Infinity**  
The largest exponent with a mantissa of 0 respresents an infinity.  
Infinity has two flavors, positive and negative. 1.0/0.0 == +INF, and 1.0/-0.0 == -INF.

**NaN**  
The largest exponent with a nonzero mantissa represents a NaN
The IEEE-754 standard does not define exactly what the mantissa has to be, and this is system-specific (other than being nonzero, otherwise it would be infinity instead)

*While there are a TON of representations for NaN, they all signify the same number (which, ironically, compares inequal to everything, including itself).*

## Single Precision (32-bits) [2,5]
```
1b    Sign      s
8b    Exponent  e
23b   Mantissa  m
```
value = (-1^s)(2^(e-127))(m/8388608)
```
Hex         | Value           | Notes
------------------------------------

      Positive Numbers
0x0000 0000 | (d) 0.0
0x0000 0001 | (d) 1.4E-45
0x3f80 0000 |     1.0
0x3f80 0001 |     1.0000001
0x4000 0000 |     2.0
0x4000 0001 |     2.0000002   | Note that with every increment of the exponent the precision is halved
            |                 | (larger numbers are less precise!)
0x4080 0000 |     4.0
0x4080 0001 |     4.0000005   | Okay, so the precision is even (slightly) less than halved, yay rounding errors!

      Negative Numbers           
0x8000 0000 | (d) -0.0
0x8000 0001 | (d) -1.4E-45
0xbf80 0000 |     -1.0
0xbf80 0001 |     -1.0000001
0xc000 0000 |     -2.0

      Special Numbers
0x7f80 0000 | +Infinity
0xff80 0000 | -Infinity
0x7f80 0001 | NaN             | All of these represent the same special number
0xff80 0001 | NaN             | All of these represent the same special number
0x7f80 0011 | NaN             | All of these represent the same special number
0x7f8f ffff | NaN             | All of these represent the same special number
```
*values with (d) signify denormalized values*

## Double Precision (64-bits) [3] ##
```
1b    Sign
11b   Exponent
52b   Mantissa
```
value = (-1^s)(2^(e-1023))(m/4503599627370496)

*I'm not going to do this table, because it's big, it's the exact same premise as 32-bits, and I'm lazy.*

## Comparison Techniques [4]

This section discusses various techniques used for comparing two floating point numbers for equality.

**Direct equality (a == b)**  
Due to rounding errors, this method is practically useless. For example, (1/3)*3, and 1.0 will probably not compare equal.

**Tolerance (fabs(a-b) <= MY_TOLERANCE)**  
This is the simplest technique, and ironically is the only one in this list that actually works all of the time. Unfortunately, it also requires prior knowledge of the data to determine an appropriate tolerance, when this isn't possible, keep reading...

**Epsilon (fabs(a-b) <= std::numeric_limits::epsilon)**  
One common technique to compare floats is to compare with epsilon, which is the difference between 1.0 and the next largest number. This method is more useful than direct equality, however it still has many faults. Since floats below 1.0 will have more precision than epsilon, this number becomes fairly useless and this method will return false positives. Numbers above 1.0 have the opposite problem where the distance between them gets bigger and this method returns false negatives as epsilon becomes too small to compare. Attempting to scale epsilon also subjects you to more rounding errors, which solves nothing.

**ULPs**  
The ULPs between two numbers are the Units of Least Precision between two numbers. If there are 2 representable numbers between them, there are 3ULPs of precision. Comparing two numbers to make sure they are within a given number of ULPs is usually more costly, but it can be much more effective. This method fails entirely however when comparing across signs, and also requires intimate knowledge of float-int conversion to implement properly and efficiently.

## References

1. IEEE-754                             https://en.wikipedia.org/wiki/IEEE_754
2. IEEE-754 Single Precision            https://en.wikipedia.org/wiki/Single-precision_floating-point_format
3. IEEE-754 Double Precision            https://en.wikipedia.org/wiki/Double-precision_floating-point_format
4. Comparing Floating Point Numbers     https://bitbashing.io/comparing-floats.html

## Other Resources
Hex/Binary/Decimal Converter         https://www.h-schmidt.net/FloatConverter/IEEE754.html
