---
title: "Data Transmission with Delphi (Part 2: Arrays and Pointer Math)"
excerpt_separator: "<!--more-->"
categories:
  - DLL
  - Records
  - Structs
  - Pointer Math
  - Arrays
tags:
  - Delphi  
  - Communication
  - Cross-Platform
---

In this blog post I will continue with arrays in communicating cross-platform via DLLs. 

<!--more-->
The purpose of this blog series is to understand how to transfer raw data at high speed with direct access using pointers to structures that can be supported by most programming platforms.


## Arrays ##

Arrays are contiguous blocks of items of the same size. In Delphi arrays can be declared explicitly in a few ways

### Fixed Length Arrays ###

If we know the size of the array of data we need to declare, or if the size never varies, then the array can be declared as follows:

{% highlight pascal %}
var
  LStaticArray : Array[0..9] of Double;
{% endhighlight %}  

A block of memory will now exists consisting of 10 doubles in a row. The adress of a static array is the same as the address of its first element. This means `@LStaticArray` and `@LStaticArray[0]` return the same value;

{% highlight pascal %}
Assert(@LStaticArray = @LStaticArray[0]); //Memory address of static array is same as address of the first element
{% endhighlight %}  

If we call `SizeOf` on the static array we get the size of all the elements as a whole. 
{% highlight pascal %}
Assert(SizeOf(LStaticArray) = SizeOf(LStaticArray[0]) * Length(LStaticArray));
{% endhighlight %}  

The size is the same size as the strucutre in memory, because the distance from element to element (stride) is simply the size of the element type. 

{% highlight pascal %}
Assert(NativeUInt(@LStaticArray[1]) - NativeUInt(@LStaticArray[0]) = SizeOf(LStaticArray[0]));
{% endhighlight %}  

If we assign one fixed array to another we copy by value. The array does not move around in memory but stays at its initial location. Assignment simply copies the entire memory content.

{% highlight pascal %}
type
  TTenDoubles =  Array[0..9] of Double;
var
  LStaticArray : TTenDoubles;
  LStaticArray2 : TTenDoubles;
begin  
  LStaticArray2[0] := '123.456';
  	
  LStaticArray :=  LStaticArray2;
  Assert(LStaticArray[0] = LStaticArray2[0]);  // values match
  Assert(@LStaticArray[0] <> @LStaticArray2[0]); // addresses do not!
end;
{% endhighlight %}  

*Note:* We had to declare a type to allow assignment, if you'd like to understand why then read about the rules of [assignment compatibility](http://docwiki.embarcadero.com/RADStudio/Rio/en/Overloads_and_Type_Compatibility_in_Generics). 
{: .notice}

When we assign a record, the data contained within the size of the record's definition is copied, but the memory locations remain the same. Static arrays and record structures can have an overhead due to copy by value. If we were to declare a method that takes our fixed length array or record as a parameter, we should usually declare the parameter as `var` or `const`.

{% highlight pascal %}
procedure foo(ADblArray: TTenDoubles);
begin
// When I was called, I copied all the values passed into ADblArray
end;

procedure goo(var ADblArray: TTenDoubles);
begin
// I hold a reference to the original record passed via ADblArray
end;
{% endhighlight %}  

If we declare a fixed array in our record it will contribute to the size of the record. The memory will be contained within the record structure. This provides for simple memory management. 

### Dynamic Arrays ###

Dynamic arrays are declared without specifying the number of elements
{% highlight pascal %}
var
  LDynamicArray: TArray<Double>;
  LDynamicArray2: Array of Double;
{% endhighlight %} 

Even though these two arrays are the same, I recommend that you use the generic syntax because it allows us to assign arrays of the same type. Dynamic arrays are allocated by calling `SetLength()`
  
{% highlight pascal %}
var
  LDynamicArray: TArray<Double>;
begin
  SetLength(LDynamicArray, 10); //allocate an array of 10 doubles
end;
{% endhighlight %} 

Like our static array the distance between elements is the size of the elements

{% highlight pascal %}
Assert(NativeUInt(@LDynamicArray[1]) - NativeUInt(@LDynamicArray[0]) = SizeOf(LDynamicArray[0])); 
{% endhighlight %}  

However, the memory address of dynamic array is not the same as address of the first element:

{% highlight pascal %}
Assert(@LDynamicArray <> @LDynamicArray[0]);
{% endhighlight %}  

If we change our dynamic array size we will find that the pointer to the dynamic array stays the same, while the pointer to the data (the raw memory array), may change. `SetLength` also copies the memory of the array to the newly allocated array as needed. 

{% highlight pascal %}
var
  LDynamicArray: TArray<Double>;
  lPtrArray: Pointer;
  LPtrArrayData: Pointer;
  i: integer;
begin

  SetLength(LDynamicArray, 10);
  lPtrArray := @LDynamicArray;
  LPtrArrayData := @LDynamicArray[0];
  
  for i := 0 to 9 do
    LDynamicArray[i] := Double(i * 1.2);
  
  SetLength(LDynamicArray, 100);
  
  Assert(lPtrArray = @LDynamicArray); // Memory address of dynamic array is the same after SetLength
  Assert(LPtrArrayData <> @LDynamicArray[0]); // The raw data array moved
  
  for i := 0 to 9 do
    Assert(LDynamicArray[i] = Double(i * 1.2));  // The values of the first 10 elements remain the same
	
  LPtrArrayData := @LDynamicArray[0];
  SetLength(LDynamicArray, 5);
  
  Assert(lPtrArray = @LDynamicArray); // Memory address of dynamic array is the same after SetLength
  Assert(LPtrArrayData <> @LDynamicArray[0]); // The raw data array moved
  
  for i := 0 to 4 do
    Assert(LDynamicArray[i] = Double(i * 1.2));  //the remaining 5 elements remained the same
  	  
end;
{% endhighlight %}   

*Note:* When calling `SetLength` we may get the same memory address for our raw data, usually when shrinking the array. You may need to grow or shrink the array by many elements to force a new address. 
{: .notice}

Assigning a dynamic array has some interesting consequences that can cause some unexpected bugs. Each dynamic array has its own address, but once we assign a dynamic array the raw data is shared between the two. As soon as we call `SetLength()` on these arrays they may diverge and become independent again.

{% highlight pascal %}
var
  LDynamicArray: TArray<Double>;
  LDynamicArray2: TArray<Double>;
  i: integer;
  V: double;
begin
  try
    SetLength(LDynamicArray, 10);
    LDynamicArray2 := LDynamicArray;
    Assert(@LDynamicArray2 <> @LDynamicArray); // The Dynamic arrays have different addresses

    Assert(@LDynamicArray2[0] = @LDynamicArray[0]); // But their raw data exists in the same location!

    for i := 0 to 9 do
    begin
      V := i * 1.2;
      LDynamicArray[i] := V;         // modifying the elements of one array
      Assert(LDynamicArray2[i] = V); // affects the other
      Assert(@LDynamicArray2[i] = @LDynamicArray[i]); // because the data exists in the same memory location  
    end;
end;
{% endhighlight %}   

The benefit of this behavior is that we can have large amounts of data passed around using dynamic arrays. Dynamic arrays are reference counted and automatically disposed. Any API that uses the raw data pointer in a dynamic array must ensure that:
- The dynamic array is referenced for as long as we need a pointer to the raw data
- The pointer to the raw data needs to be updated after `SetLength` is called against the dynamic array

### Records with Variable Length Arrays ###

We can declare records that hold arrays where we use the syntax of a fixed length array (typically with a single element), but where the actual size of the array varies.

In Part 1 we encountered one of these:

[DEV_BROADCAST_DEVICEINTERFACE](https://docs.microsoft.com/en-us/windows/desktop/api/dbt/ns-dbt-_dev_broadcast_deviceinterface_w). Which is defined as

{% highlight C %}
typedef struct _DEV_BROADCAST_DEVICEINTERFACE_W {
  DWORD   dbcc_size;
  DWORD   dbcc_devicetype;
  DWORD   dbcc_reserved;
  GUID    dbcc_classguid;
  wchar_t dbcc_name[1];
} DEV_BROADCAST_DEVICEINTERFACE_W, *PDEV_BROADCAST_DEVICEINTERFACE_W;
{% endhighlight %}  

Which I translated to Delphi as

{% highlight pascal %}
PDEV_BROADCAST_DEVICEINTERFACE = ^DEV_BROADCAST_DEVICEINTERFACE;
DEV_BROADCAST_DEVICEINTERFACE = record
  dbcc_size: DWORD;
  dbcc_devicetype: DWORD; // = DBT_DEVTYP_DEVICEINTERFACE
  dbcc_reserved: DWORD;
  dbcc_classguid: TGUID;
  dbcc_name: Char;
end;
{% endhighlight %}  

Strictly speaking, my direct conversion of `dbcc_name` should have been declared as `dbcc_name: array[0..0] of Char;`, a fixed length array of one character! Well, the actual name of the device is actually longer than just one character. The record's size as described in `dbcc_size` includes the length of the string contained in `dbcc_name` including the null character. 

In Delphi `pChar` is a pointer to `Char`. We can read characters beyond that first character by indexing (e.g. `LMyPChar[5]`). Delphi provides support to convert from `pChar` to `String`. When a variable of type `pChar` is converted to `String` the characters are read until a null character (character with value 0) is reached. This means that we can read a null terminated character array without knowing its size, or in this case without reading the size of the record. 

{% highlight pascal %}
// WMDeviceChange sample in lesson1
 LUsbDeviceName := PChar(@LPBroadcastDeviceIntf^.dbcc_name);
{% endhighlight %}  
		  
**Watch out!** Assigning a variable of type `DEV_BROADCAST_DEVICEINTERFACE` will transfer the `dbcc_size`, but only the first character of `dbcc_name`.
{: .notice--warning}		  
		  
Typically records such as these are used in APIs where a single record is passed. Since the size of `DEV_BROADCAST_DEVICEINTERFACE` varies we cannot place it in an array. We can place a series of these records in a row, but indexing will not work because we do not have a fixed stride length between elements. In that case we would have to move our pointer by the size of each record in turn. 

To create a record such as this you would allocate memory equal to the size of the record definition plus the length of the character string. That size value would then also be written to the `dbcc_size` field.

## Pointer Math ##

We can pass a pointer to the first element of our array to our DLL, we can also pass the number of elements. If we know what stride to take from the start of one item to the next, then we can walk our pointer by simply adding (or deleting) the size of this stride to the numeric value of the pointer. As we walk our pointer and de-reference a memory address, we can read the items in our array. 

The question then is, what is the stride between my elements? Can we safely use the size of our record? 

In Part 1 we saw that our records are aligned to memory boundaries, but the record may also be padded at its end. If another record of the same type is now placed in memory directly after the first, then it's starting address will be correct, and no alignment is needed. This end padding counts towards the size of the record, which means that we can use the size of the record as our stride from one element to the next. 

### Inc and Dec ###

Calling `Inc` and `Dec` increments and decrements our pointer's value by the size of the pointer's underlying type. 

{% highlight pascal %}
type
  PTestRec = ^TTestRec;
  TTestRec = Record
    Field1: word;
    Field2: byte;
  End;

var
  LTestPtr: PTestRec;
  
begin
  LTestPtr := nil; // nil has an integer value of 0
  Inc(LTestPtr);
  Assert(NativeUInt(LTestPtr) = SizeOf(TTestRec));
  Dec(LTestPtr);
  Assert(LTestPtr = nil);
end;
{% endhighlight %}  

As we call `Inc` on our pointer starting at the first element it will simply advance and reference element by element. 

{% highlight pascal %}
var
  LTestPtr: PTestRec;
  LTestArray : Array[0..5] of TTestRec;
  i: integer;
begin
 LTestPtr := @LTestArray[0];

 for i := 0 to 5 do
 begin
   Assert(LTestPtr = @LTestArray[i]);
   Inc(LTestPtr);
 end;
{% endhighlight %} 

So, with a strongly typed pointer and the count we can easily access all of our elements in the array in a sequential fashion. 

### Treat the Pointer as a number ###
To avoid the `Inc` and `Dev` has side effect of modifying the pointer, we can also do the math ourselves. All we must do is multiple the size of the element with the number of elements to walk and then add to our starting point.

{% highlight pascal %}
var
  LTestPtr: PTestRec;
  LTestArray : Array[0..5] of TTestRec;
  i: integer;
begin
  LTestPtr := @LTestArray[0]; //starting point

  for i := 0 to 5 do
  begin
    Assert( PTestRec(NativeUInt(LTestPtr) + i *SizeOf(TTestRec)) = @LTestArray[i]);
  end;
end;
{% endhighlight %} 

There is a shorthand for this notation

### Pointer Indexing ###

Writing `PTestRec[i]` is essentially the same as `PTestRec(NativeUInt(LTestPtr) + i *SizeOf(TTestRec))^` and allows us to use the same indexing notation as we would use with arrays. This notation works with a few built-in types, most notably `PChar`, but we can use the `{$POINTERMATH ON}` directive to allow indexing on our own pointer types

{% highlight pascal %}
var
  LTestPtr: PTestRec;
  LTestArray : Array[0..5] of TTestRec;
  i: integer;
begin
  LTestPtr := @LTestArray[0]; // our pointer reference the first element in our array

{$POINTERMATH ON}
   for i := 0 to 5 do
   begin
     Assert(@LTestPtr[i] = @LTestArray[i]); // we index based on our pointer in the same way as the array
   end;

  // Modifying via our array or using our indexed pointer modifies the same data 
  LTestArray[2].Field1 := 12;
  Assert(LTestPtr[2].Field1 = 12);

  LTestPtr[3].Field2 := 34;
  Assert(LTestArray[3].Field2 = 34);
{$POINTERMATH OFF}  
end;

{% endhighlight %} 

Most of you would have indexed `PChar`. The index syntax here is the same, but just expanded for our own pointer types. `PChar` is different from most other data pointers in that it usually does not need to be paired with a size. A `PChar` is usually terminated by a null character. We can potentially index data beyond the end of our array with other pointer types and we need to have the element count passed via our API. With the count and a pointer to a known structure type (or some way to derive this information), we can read data in arrays in most programming languages


## Section Conclusion ##
Arrays are contiguous blocks of same size values. Pointers to the raw data of arrays can be used to traverse the elements in an array. To successfully traverse an array in our API we need a pointer to the start of the data of the array, the size of the elements and the number of elements to read.

In the next section I will cover the basic API of our DLL that will allow it to receive records and arrays.

