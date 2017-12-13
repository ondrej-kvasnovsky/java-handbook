# Binary Representation of Numbers

First lets see how binary numbers are created. For positive numbers, 

0 == 0000 0000  
1 == 0 + 1 == 0000 0001  
2 == 1 + 1 == 0000 0010  
3 == 2 + 1 == 0000 0011  
4 == 3 + 1 == 0000 0100  
The other way around, 2 == 3 - 1, 1 == 2 - 1, and 0 == 1 - 1.

Now we come to -1. That's obviously 0 - 1. However, there's nothing to subtract from 0. So they came up with the special notation of all ones: 1111 1111. So -1 == 1111 1111. Let's continue this way for negative numbers:  
-2 == -1 - 1 == 1111 1110  
-3 == -2 - 1 == 1111 1101  
-4 == -3 - 1 == 1111 1100



