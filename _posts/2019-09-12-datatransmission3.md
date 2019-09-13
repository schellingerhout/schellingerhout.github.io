---
title: "Data Transmission with Delphi (Part 3: Data at Sender and Receiver)"
excerpt_separator: "<!--more-->"
categories:
  - Data Transmission
tags:
  - Delphi  
  - Data Transmission
  - Cross-Language
  - DLL
  - Records
  - Structs
  - Pointer Math
  - Arrays  
---

In this blog post I will continue with arrays in communicating cross-platform via DLLs. 

<!--more-->
The purpose of this blog series is to understand how to transfer raw data at high speed with direct access using pointers to structures that can be supported by most programming platforms.


## Records can interpret data in different ways ##
As mentioned in prior sections that pointers are simply numbers that correspond to places in memory and that types associated with that address are the way we interpret the data. In many cases data can be interpreted in the same way. Here is an example of we could define a customer record at a transmitter:


{% highlight pascal %}
type

  {$Z4} //integer enum
  TxRectTypeEnum = (TxRectType_AccountHolder,  TxRectType_CustomerInfo);
  {$Z1}

  TxCustomerInfoRec = Record
    Size: Cardinal;
    RecType: TxRectTypeEnum;

    UID: TGUID;
    ID: integer;

    Name, 
      Address1, 
      Address2 : String; 
  End;

{% endhighlight %}  

We can interpret the same data received as a pointer in a generic way at the receiver as follows

{% highlight pascal %}
type

  RxCustomerInfoRec = Record
    Size: Cardinal;
    RecType: Integer;

    UID: TGUID;
    ID: integer;

    Name, 
        Address1, 
        Address2 : PChar;
  End;

{% endhighlight %}  

`RecType` can be interpreted as a `Integer` and the `String` can be interpreted as `PChar`. Similarly to the `String` field we could interpret a type of `TArray<Double>` as `PDouble`;

## Who manages the memory? ##

In the scenario described above the sender holds the record that has the `String` type. This type just like dynamic arrays (like `TArray<Double>`) are reference counted types. Referencing the underlyping data of these types as either `PChar` for `String`, or `PDouble` for `TArray<Double>` does not increment the reference count. This means that as soon as the dynamic array or string is out of scope the memory is disposed and the pointer is invalid.

*Note:* Dynamic arrays and strings must be kept alive via a reference as long as there is a pointer to their underlying data.
{: .notice}

There are a few ways to manage this. 
    * Receiving party allocates memory for the transmitting party to fill with data. The receiving party disposes the memory after it processed it (and at its convenience).
    * Transmitting party allocates memory for the receiving party to copy or process completely (no references held). The transmitting party disposes the memory 
        * once the send method returns, or 
        * waits to be notified that the receiving party is done with the data, or
        * the receving party terminates the session

The scenario of allocating memory to be filled is very common in the Windows API, this is often a two step call. The first call would pass the buffer as `nil` and receive the size that you need to allocate.

In the rest of this discussion I will focus on the easier method of synchronous transmission where I assume data is received and copied or completely processed into other types at the receiver before I return. This way the transmitter can dispose as soon as the method call into the receiver returns


## Directing incoming data ##



## Section Conclusion ##
