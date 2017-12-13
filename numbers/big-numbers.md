# Big Numbers

We are going to look at how to work with big number. Bigger than what can fit into primitive types like `int` and `long`.

## Integers

Normally, we work with primitive types. Here is an example loop that will try to verify if a number is prime number.

```
boolean isPrime = true
int candidate = 999
for (int i = 2; i < candidate - 1; i++) {
  int remainder = candidate % i;
  if (remainder == 0) isPrime = false
}
println isPrime
```

How would the same code look like using BigInteger, so we can check whether a very very very big number is prime number?

```
boolean isPrime = true;
BigInteger candidate = new BigInteger("999");
for (BigInteger i = BigInteger.valueOf(1); i.compareTo(candidate) < 0; i = i.add(BigInteger.ONE)) {
  BigInteger remainder = candidate.remainder(i);
  if (remainder == 0) isPrime = false;
}
println isPrime;
```

> Try to experiment, try to put there really big number and find the biggest prime number \(you might need plenty of time to do that\). You can try to verify if 1001000000000000000000000000000000023 is a prime number ;-\)



