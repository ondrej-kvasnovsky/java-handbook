# Operators

Simple summary of operators in Java.

## Bitwise operators

### AND 

AND operator produces `1` only if both numbers are `1`. So, `1 | 2` should produce `0`. On the other hand, `1 | 3` should print `1`, becuase they will have one `1` in common \(then 1 in the middle of that binary number\). 

```
int one = 1; // is 001 in binary
int two = 2; // is 010 in binary
int three = 3; // is 011 in binary

int and = one & two;
System.out.println(and); // prints out 0

one &= three;
System.out.println(one) // prints out 1
```

### OR

`001 OR 010` will result in `011`, which is 3. Also `011 OR 011` will result in the same number. Because you need to take each binary number and apply OR. Lets take 1 and 0 from the first example, that are on zeroth position, 001 and 010, `1 OR 0` results in 1. `1 OR 1` results in 1. `0 OR 1` results in 1.

```
int one = 1; // is 001 in binary
int two = 2; // is 010 in binary
int three = 3; // is 011 in binary

int or = one | two; 
System.out.println(or); // prints out 3

or |= three;
System.out.println(or);// prints out 3
```



