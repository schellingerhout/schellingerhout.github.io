---
title: "Implementing the Sieve of Eratosthenes in Delphi"
excerpt_separator: "<!--more-->"
categories:
  - Algorithms
tags:
  - Delphi
  - Prime Numbers
---

The Sieve of Eratosthenes is a very fast, but memory intensive algorithm to find the primes under a given value. 

<!--more-->

I first read about this method when I was about 12 in a book called "Mathematics: a Human Edeavor" by Harold R. Jacobs. I had forgotten about the details of this method until I stumbled on it again the other day. Here is how I developed a solution in Delphi:

## The algortihm

Start by creating a field of numbers from 3 up to desired maximum. 

**3,  4,  5,  6,  7,  8,  9,  10,  11,  12,  13,  14,  15,  16,  17,  18,  19,  20,  21,  22,  23,  24,  25,  26,  27,  28,  29,  30,  31,  32,  33,  34,  35,  36,  37,  38,  39,  40,  41,  42,  43,  44,  45,  46,  47,  48,  49,  50,  51,  52,  53,  54,  55,  56,  57,  58,  59,  60,  61,  62,  63**

Mark multiples of 2 starting with 2<sup>2</sup> 

**3**,  <s>4</s>,  **5**,  <s>6</s>,  **7**,  <s>8</s>,  **9**,  <s>10</s>,  **11**,  <s>12</s>,  **13**,  <s>14</s>,  **15**,  <s>16</s>,  **17**,  <s>18</s>,  **19**,  <s>20</s>,  **21**,  <s>22</s>,  **23**,  <s>24</s>,  **25**,  <s>26</s>,  **27**,  <s>28</s>,  **29**,  <s>30</s>,  **31**,  <s>32</s>,  **33**,  <s>34</s>,  **35**,  <s>36</s>,  **37**,  <s>38</s>,  **39**,  <s>40</s>,  **41**,  <s>42</s>,  **43**,  <s>44</s>,  **45**,  <s>46</s>,  **47**,  <s>48</s>,  **49**,  <s>50</s>,  **51**,  <s>52</s>,  **53**,  <s>54</s>,  **55**,  <s>56</s>,  **57**,  <s>58</s>,  **59**,  <s>60</s>,  **61**,  <s>62</s>,  **63**

After this we repeatedly find the lowest unmarked value (which will always be prime) and mark its multiple starting with its square

Mark multiples of 3 starting with 3<sup>2</sup> 

**3**,  <s>4</s>,  **5**,  <s>6</s>,  **7**,  <s>8</s>,  <s>9</s>,  <s>10</s>,  **11**,  <s>12</s>,  **13**,  <s>14</s>,  <s>15</s>,  <s>16</s>,  **17**,  <s>18</s>,  **19**,  <s>20</s>,  <s>21</s>,  <s>22</s>,  **23**,  <s>24</s>,  **25**,  <s>26</s>,  <s>27</s>,  <s>28</s>,  **29**,  <s>30</s>,  **31**,  <s>32</s>,  <s>33</s>,  <s>34</s>,  **35**,  <s>36</s>,  **37**,  <s>38</s>,  <s>39</s>,  <s>40</s>,  **41**,  <s>42</s>,  **43**,  <s>44</s>,  <s>45</s>,  <s>46</s>,  **47**,  <s>48</s>,  **49**,  <s>50</s>,  <s>51</s>,  <s>52</s>,  **53**,  <s>54</s>,  **55**,  <s>56</s>,  <s>57</s>,  <s>58</s>,  **59**,  <s>60</s>,  **61**,  <s>62</s>,  <s>63</s>

The next unmarked value greater than 3 is 5. Mark multiples of 5 starting with 5<sup>2</sup>

**3**,  <s>4</s>,  **5**,  <s>6</s>,  **7**,  <s>8</s>,  <s>9</s>,  <s>10</s>,  **11**,  <s>12</s>,  **13**,  <s>14</s>,  <s>15</s>,  <s>16</s>,  **17**,  <s>18</s>,  **19**,  <s>20</s>,  <s>21</s>,  <s>22</s>,  **23**,  <s>24</s>,  <s>25</s>,  <s>26</s>,  <s>27</s>,  <s>28</s>,  **29**,  <s>30</s>,  **31**,  <s>32</s>,  <s>33</s>,  <s>34</s>,  <s>35</s>,  <s>36</s>,  **37**,  <s>38</s>,  <s>39</s>,  <s>40</s>,  **41**,  <s>42</s>,  **43**,  <s>44</s>,  <s>45</s>,  <s>46</s>,  **47**,  <s>48</s>,  **49**,  <s>50</s>,  <s>51</s>,  <s>52</s>,  **53**,  <s>54</s>,  <s>55</s>,  <s>56</s>,  <s>57</s>,  <s>58</s>,  **59**,  <s>60</s>,  **61**,  <s>62</s>,  <s>63</s>

7 is the next unmarked value. So we mark multiples of 7 starting at 7<sup>2</sup>

**3**,  <s>4</s>,  **5**,  <s>6</s>,  **7**,  <s>8</s>,  <s>9</s>,  <s>10</s>,  **11**,  <s>12</s>,  **13**,  <s>14</s>,  <s>15</s>,  <s>16</s>,  **17**,  <s>18</s>,  **19**,  <s>20</s>,  <s>21</s>,  <s>22</s>,  **23**,  <s>24</s>,  <s>25</s>,  <s>26</s>,  <s>27</s>,  <s>28</s>,  **29**,  <s>30</s>,  **31**,  <s>32</s>,  <s>33</s>,  <s>34</s>,  <s>35</s>,  <s>36</s>,  **37**,  <s>38</s>,  <s>39</s>,  <s>40</s>,  **41**,  <s>42</s>,  **43**,  <s>44</s>,  <s>45</s>,  <s>46</s>,  **47**,  <s>48</s>,  <s>49</s>,  <s>50</s>,  <s>51</s>,  <s>52</s>,  **53**,  <s>54</s>,  <s>55</s>,  <s>56</s>,  <s>57</s>,  <s>58</s>,  **59**,  <s>60</s>,  **61**,  <s>62</s>,  <s>63</s>

11 is the next prime, but 11<sup>2</sup> is outside our max. We are done and all unmarked values are prime. You can see how this algorithm can be very quick esp. since we only have to mark prime multiples and then only multiples greater than the square of each prime in turn.

## What is the downside to this method?

We need at least one bit of storage per number that we want to mark. For simplicity if our bitfields start at 0 (and I'll show why that is a good idea soon) and we want to mark primes up to max(uint32) 4,294,967,295. Then we would need 512 MB just for the bits. We could shrink the bitfield by only storing the odd values and starting from 3.

The algorithm is very fast, but the memory constraints make it problematic to get large primes.

Supposing that we will need to use memory for other purposes, such as storing the prime factorization of numbers we should consider building and running the application as a 64-bit binary to allow easy access to more memory. 

## Implementation in Delphi

### The Bitfield

For the purposes of keeping the implementation simple and still gaining speed I will make the bitfield starting at 0 and a bit for each value. Index 0 represents 0, index n represents n. This keeps the logic very simple. I simply index by each value directly and set the bit. But, what if we want to store values larger than our sample bit-field of 64 bits? We could simply keep an array of 64-bit fields

for example we could have `FBitFieldArray : Array[0..2] of UInt64;` to represent 0 to 191

`FBitFieldArray[0]` would be 0 to 63, `FBitFieldArray[1]` would be 64 to 127, `FBitFieldArray[2]` would be 128 to 191. Indexing on field [0] is easy, we just use our number, but what about the other fields. In Delphi integer division is done with `Div`, while the `Mod` method gets the remainder (modulus) of a number after division. Both can be done at the same time with `DivMod`. Since we have blocks of 64 we can divide by 64 to get the block we are in and the remainder would be the position inside that block

 {% highlight pascal %}
DivMod(index, 64, LBlockIndex, LBlockBitIndex);
{% endhighlight %}

We will build our code based on 3 blocks and then we will generalize later 

### 2 is the odd man out

All primes are odd except for 2, which forces us to have a check every iteration of the loop for this exceptional case. In all other cases we can check for our next prime value starting with the previous prime's position plus 2.

To eliminate this the check for 2 each iteration (and since 2 is a trivial case) we can set all the bits in our bitfield for 2s first before repeating the same process for primes 3 and up.

 {% highlight pascal %}
LMultiple := 2;
LNonPrime := 4; //start at 2^2
while LNonPrime <= 191 do // maximum value represented by block [2]
begin
  DivMod(LNonPrime, 64, LBlockIndex, LBlockBitIndex);
 
  // Set bit LBlockBitIndex in FValueBitField[LBlockIndex] to 1 

  inc(LMultiple);

  LNonPrime := 2 * LMultiple;
end;
{% endhighlight %}

But, this is not needed. We know that setting every even value will just yield a bit pattern of alternating ones and zeros 101010101. We do have a special case though; 2 is not prime and, of course, 0 and 1 are not prime numbers either. Since binary 1010<sub>2</sub> is just hexadecimal 5<sub>16</sub> we can set all blocks to `$5555555555555555` except for the very first block. The only difference for the first block is that the first 4 bits would be 011<sub>2</sub> or 3<sub>16</sub>. So, we can set the first block to `$5555555555555553`

 {% highlight pascal %}
FValueBitField[0] := $5555555555555553;
FValueBitField[1] := $5555555555555555;
FValueBitField[2] := FValueBitField[1];
{% endhighlight %}
   
Now we can move on to real algorithm.

### Eliminate multiples of primes

Starting with prime 3 we can now mark multiples of 3 as not prime. We have to exclude 3 itself of course, but we already covered 2\*3 so we don't need to do 3\*2 as well. Therefore we can start at 3*3. This logic applies as we move to each next prime, so we can always start with multiples of our prime starting with its square

 {% highlight pascal %}
LPrime := 3;
repeat
  LMultiple := LPrime;
  LNonPrime := LPrime * LMultiple;
  while LNonPrime <= FMaxValue do
  begin
    DivMod(LNonPrime, 64, LBlockIndex, LBlockBitIndex);
  
    // Set bit LBlockBitIndex in  FValueBitField[LBlockIndex]

    inc(LMultiple);
    LNonPrime := LPrime * LMultiple;
  end;
  
  // Set LPrime to the next prime
until (LPrime >= 14); // 14 ^ 2 is greater than 191
{% endhighlight %}

### Setting the bit corresponding to a composite number

Next we need to find a way to set a specific bit by index. `LBlockBitIndex` tells us the bit we need to set, but how do we actually set a bit based on that index? In numeric terms each bit n represents 2<sup>n</sup>. Therefore 2<sup>LBlockBitIndex</sup>, but we don't actually need to calculate a power of 2, we can use bit shifting. The `shl` method in Delphi shifts bits left. For instance if we start with 10000<sub>2</sub> and shift 2 left we'd get 1000000<sub>2</sub>. Using `1 shl LBlockBitIndex` we will shift the value by the number of bits. At first glance this looks OK, but what size is the `1` constant used in that expression? Its unlikely to be 64-bit, which means that shifts over the size of the `1` constant will give us incorrect results.  To ensure that we shift for a 64-bit unsigned buffer value we can make it explicit by casting `UInt64(1) shl LBlockBitIndex` 

**Watch out!**  Don't assume the sizes of declared constants either used in expressions or declared with a const statement. If you need a specific size, make it explicit
{: .notice--warning}

Now that we have a value with the bit all we need to do is to apply a bitwise `or` with the correct block in the field. This way we will set the bit at that index and leave all other bits in the field as they were before

 {% highlight pascal %}
FValueBitField[LBlockIndex] := FValueBitField[LBlockIndex] or
        (UInt64(1) shl LBlockBitIndex);
{% endhighlight %}

Delphi uses the `or` operator for logical and bitwise `or` operations. The type of operation depends on the operands, in this case we have a bitwise logical operation.

### Finding the next prime

The sieve eliminates primes by marking multiples so if we start with our last prime and move bit by bit until we find an unmarked bit then that bit will represent our next prime. So given we our field how do we do this?

We know that we have checked all values up to p (our last prime). p is 3 or larger so we know we can check for even unmarked bits p+2 or higher. In a loop for n we can shift right by p+n+2 (n > 0) until the right most bit is 0.

Of course as we increment to get the next prime we may loop over to the next block

 {% highlight pascal %}
class function TSieveOfEratosthenes.GetNextPrime(APreviousPrime
  : uint32): uint32;
var
  LBlock: UInt64;
  LBlockIndex: UInt64;
  LBlockBitIndex: UInt64;
begin
  DivMod(APreviousPrime + 2, 64, LBlockIndex, LBlockBitIndex);
  repeat
    LBlock := FValueBitField[LBlockIndex] shr LBlockBitIndex;
    if LBlockBitIndex >= 64 then
    begin
      LBlockBitIndex := 0;
      inc(LBlockIndex);
    end
    else
      inc(LBlockBitIndex);
  until (LBlock and 1 = 0);
  result := LBlockIndex * 64 + LBlockBitIndex - 1;
end;
{% endhighlight %}

This code is not fully optimized, but shows the process of finding the next unset bit. Lastly since we convert the Block and Bit index to a number which will be our next prime (we subtract 1 because our bitindex is one out of phase).

### Generalizing the sample

First lets remedy the fact that we use "magic numbers" all over the place. While it is fine to write comments to explain the use of numbers that are hard to derive from context. Instead of using 64 for our block size we can declare a constant

 {% highlight pascal %}
const
  BlockSize = 64;
{% endhighlight %}

We make assumptions that the number of blocks are fixed. That may not be the case, we want to make the sieve more flexible and allow it to expand according to demands

Let start by defining `MaxValue`; this will be the cap when searching for primes. To represent that value we would need `Ceil(MaxValue/BlockSize)` blocks in our field, so that leads us to `MaxBlockIndex := Floor(MaxValue/BlockSize)`. 

We know that we start marking multiples of squares of primes, therefore we don't have to check primes greater than the integer square root of `MaxValue`. This gives us `MaxValueISqrt := Trunc(Sqrt(MaxValue))`.

Since we can set all of these as dependency on a single value we can write something like this:

 {% highlight pascal %}
class procedure TSieveOfEratosthenes.SetMaxValue(const Value: uint32);
begin
  FMaxValue := Value;
  FMaxValueISqrt := Trunc(Sqrt(FMaxValue));
  FMaxBlockIndex := floor(FMaxValue / BlockSize);
  SetLength(FValueBitField, FMaxBlockIndex + 1);
end;
{% endhighlight %}

With these changes our main loop will now look more readable. Here is our main routine that creates the prime mask

 {% highlight pascal %}
class procedure TSieveOfEratosthenes.GetPrimeMask;
var
  i: uint32;
  LPrime: uint32;
  LMultiple: uint32;
  LNonPrime: UInt64;
  LBlockIndex: UInt64;
  LBlockBitIndex: UInt64;
begin
  //we could set the initial state of the blocks. since 
  // multiples of 2 are really trivial bit patterns 

  // first block has   repeat(0101) 0011
  FValueBitField[0] := UInt64($5555555555555553); 
  for i := 1 to FMaxBlockIndex do
  begin
   // other blocks have repeat(0101)
    FValueBitField[i] := UInt64($5555555555555555);
  end;
  
  LPrime := 3;
  repeat
    LMultiple := LPrime;
    LNonPrime := LPrime * LMultiple;
    while LNonPrime <= FMaxValue do
    begin
      DivMod(LNonPrime, BlockSize, LBlockIndex, LBlockBitIndex);
  
      FValueBitField[LBlockIndex] := FValueBitField[LBlockIndex] or
        (UInt64(1) shl LBlockBitIndex);

      inc(LMultiple);

      LNonPrime := LPrime * LMultiple;
    end;
    LPrime := GetNextPrime(LPrime);
  until (LPrime >= FMaxValueISqrt);
end;
{% endhighlight %}

We can also clean up our `GetNextPrime` to add in our new variables:

 {% highlight pascal %}
class function TSieveOfEratosthenes.GetNextPrime(APreviousPrime
  : uint32): uint32;
var
  LBlock: UInt64;
  LBlockIndex: UInt64;
  LBlockBitIndex: UInt64;
begin
  DivMod(APreviousPrime + 2, BlockSize, LBlockIndex, LBlockBitIndex);
  repeat
    LBlock := FValueBitField[LBlockIndex] shr LBlockBitIndex;
    if LBlockBitIndex >= BlockSize then
    begin
      LBlockBitIndex := 0;
      inc(LBlockIndex);
    end
    else
      inc(LBlockBitIndex);
  until (LBlock and 1 = 0);
  result := LBlockIndex * BlockSize + LBlockBitIndex - 1;
end;
{% endhighlight %}


### Finding Primes up to a Maximum
The process would be to mark all multiples primes using the method above up to the squareroot of the maximum. And then traverse the field and return the indexes of all the zeros. Assume the procedure that masks the bits is called `GetPrimeMask` and that `NumberOfBitsSet` is a method that returns the number of 1s in the field.


 {% highlight pascal %}
class function TSieveOfEratosthenes.GetPrimes(const AMaxValue: uint32)
  : ArrayOfUInt32;
var
  LPrime: Uint32;
  i: integer;
begin
  SetMaxValue(AMaxValue);
  GetPrimeMask;
  SetLength(result, FMaxValue - NumberOfBitsSet);
  result[0] := 2;
  result[1] := 3;
  LPrime := 3;
 
  i := 2;
  while LPrime <= FMaxValue do
  begin
    LPrime := GetNextPrime(LPrime);
    result[i] := LPrime;
    inc(i);
  end;
end;
{% endhighlight %}

### Couting bits

In order to allocate exactle enough memory to return our primes in the method above we needed to know the number of bits set. The difference between the max value and the number of bits set gives us our number of primes. I found [this nugget](https://stackoverflow.com/questions/2709430/count-number-of-bits-in-a-64-bit-long-big-integer
) that I converted from C to Delphi. All I do finally is looping through the blocks and accumulating the count

 {% highlight pascal %}
class function TSieveOfEratosthenes.NumberOfBitsSet: uint32;
var
  i: integer;
  LBitFieldCount: UInt64;
begin
  result := 0;
  for i := 0 to FMaxBlockIndex do
  begin
    LBitFieldCount := FValueBitField[i] - ((FValueBitField[i] shr 1) and
      UInt64($5555555555555555));

    LBitFieldCount := (LBitFieldCount and UInt64($3333333333333333)) +
      ((LBitFieldCount shr 2) and UInt64($3333333333333333));

    LBitFieldCount :=
      byte((((LBitFieldCount + (LBitFieldCount shr 4)) and
      UInt64($F0F0F0F0F0F0F0F)) * UInt64($101010101010101)) shr 56);

    result := result + LBitFieldCount;
  end;
end;
{% endhighlight %}


## Conclusion
I hope that you found this useful or at least interesting. Please download, clone or fork the [source code](https://github.com/schellingerhout/sieve-of-eratosthenes-delphi).