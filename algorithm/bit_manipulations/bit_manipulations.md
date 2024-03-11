# bit manipulations

https://leetcode.com/tag/bit-manipulation/

## Key Concepts
1.  two's complement representation of numbers in Java
    https://en.wikipedia.org/wiki/Two%27s_complement#Converting_to_two's_complement_representation
2.  Brian Kernighan's Algorithm
    n & (n - 1), clear the rightmost set bit.
    https://yuminlee2.medium.com/brian-kernighans-algorithm-count-set-bits-in-a-number-18ab05edca93

### Familiar with bit representation of a number
https://leetcode.com/problems/power-of-four/description/

### Bit operations
https://leetcode.com/problems/sort-integers-by-the-number-of-1-bits/description/

```
    // count # of bits set in n
    int weights = 0;
    while (n > 0) {
        weights += n & 1;
        n >>= 1;
    }
```
```
n & (n-1)  // clear the rightmost set bit.
```

###
