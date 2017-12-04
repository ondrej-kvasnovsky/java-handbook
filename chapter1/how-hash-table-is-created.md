# How Hash Table Is Created

We all might have heard that inserting, updating and obtaining values from a hash table is fast. It should be 0\(1\), because the elements are accessed by its hash. If there is a conflict in hashes, it starts creating linked list on that has position for all the values with the same hash \(this should be considered as an error, but anyway, hash table implements that\).

Basic data structure is array and we know that array is used to in hash table as well.

How is the hash calculated so it makes sure it fits into an array of certain size? Because when we work with arryas, we do not care about hashes, we only care about indexes.

Lets take a simple example to elaborate on that.

## Hashtable

Lets say we want to create hashtable with intial size of 20 elements. Then we try to get indexes for 10 elements.

```
int hashtableSize = 20;
String [] values = ["Hi", "How", "are", "you", "doing", "I", "am", "doing", "very", "well"];

for (String value : values) {
  println value.hashCode() & 0x7FFFFFFF % hashtableSize;
}
```

> 0x7FFFFFFF is a number in hexadecimal \(2,147,483,647 in decimal\) that represents the maximum positive value for a 32-bit signed binary integer.

Observe how many conflicts \(same indexes we get here\).

```
1
0
4
7
7
1
4
7
6
6
```

## HashMap

HastMap is using different approach, it will create initial size of up to the array as power of 2 of the provided size. Here is how it works.

```
int tableSizeFor(int cap) {
  int max = 1 << 30;
  int n = cap - 1;
  n |= n >>> 1;
  n |= n >>> 2;
  n |= n >>> 4;
  n |= n >>> 8;
  n |= n >>> 16;
  return (n < 0) ? 1 : (n >= max) ? max : n + 1;
}

println tableSizeFor(1)
println tableSizeFor(2)
println tableSizeFor(32)
println tableSizeFor(64)
println tableSizeFor(128)
println tableSizeFor(256)
println tableSizeFor(257)
println tableSizeFor(1000)
println tableSizeFor(4000)
```

When we execute this code, we always get the power of 2.

```
1
2
32
64
128
256
512
1024
4096
```

Here is a sample code that could generate indexes inside a hast map.

```
int requestedHashMapSize = 20;
int hashMapSize = tableSizeFor(requestedHashMapSize);
String [] values = ["Hi", "How", "are", "you", "doing", "I", "am", "doing", "very", "well"];
for (String value : values) {
  int sizeMinusOne = hashMapSize - 1;
  println sizeMinusOne & (value.hashCode() ^ (value.hashCode() >>> 16));
}
```

Here are the indexes. Even we have bigger size of array, we get redundant index and thus we need to create a linked list from that index. 

```
1
17
21
30
2
9
12
2
1
25
```



