# How Hash Table Is Created

We all might have heard that inserting, updating and obtaining values from a hash table is fast. It should be 0\(1\), because the elements are accessed by its hash. If there is a conflict in hashes, it starts creating linked list on that has position for all the values with the same hash \(this should be considered as an error, but anyway, hash table implements that\). 

Basic data structure is array and we know that array is used to in hash table as well. 

How is the hash calculated so it makes sure it fits into an array of certain size? Because when we work with arryas, we do not care about hashes, we only care about indexes.

Lets take a simple example to elaborate on that. 



