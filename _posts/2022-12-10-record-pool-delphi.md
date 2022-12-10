---
title: "A Dynamic Record Pool with Stable Pointers in Delphi"
excerpt_separator: "<!--more-->"
categories:
  - Memory Management
tags:
  - Delphi  
  - Pointers
  - Records
  - Memory
---

The design of a memory pool in Delphi that holds records (structures). This pool can grow, while never moving records in memory (this is also called pointer stability). 

<!--more-->

## Motivation 

One of the common requirements while working in computational geometry is the need to dynamically create or allocate memory for records.  However, heap memory operations are generally expensive and time consuming.  In contrast, memory management on the stack is extremely efficient because memory is linearly allocated and disposed.  

Unfortunately, stack memory is limited, so we can't escape our usage of the free store. However, we can make our allocations more efficient. For instance, we can allocate blocks of memory instead of individual elements.  

Many computational geometry algorithms depend on linked lists or doubly linked lists and therefore require pointer stability, meaning we cannot move records around in memory.  

## Dynamic Arrays Revisited 

Before we continue designing our record pool, first just a recap of how dynamic arrays work. For more detail on how dynamic arrays work, please reference my [prior post on data transmission.]({{ site.baseurl }}{% post_url 2019-02-14-datatransmission2 %})

Dynamic arrays are smart pointers to the element type declared in their definition. For instance, `TArray<TMyRecord>` is also a record pointer of type `PMyRecord=^TMyRecord`. The difference is that the dynamic array also stores the reference count and the number of elements right before the very first element in the array. This means our memory looks like this: 

```
Reference Count, Length, Element0, Element1, … 
```

Our dynamic array holds the address of `Element0` and can therefore also be used as a simple pointer type. 

Every time this dynamic array enters a scope its reference count is increased, and every time it exits a scope the reference count is decreased. Once the reference count hits zero the memory of the array is disposed.  

## The Problem with Dynamic Arrays 

If we resize a dynamic array a new larger block is allocated, and the array’s memory is copied. This could be an expensive operation, but more notably the addresses of all the elements would change. This violates our requirement of pointer stability. 

If we hold our records in dynamic arrays, we cannot resize them. The most practical way to prevent consumers of our memory pool from direct access is to make the dynamic array a private field of a record or class.  

We can also expose a method that will give us the pointer to the next element in sequence. To maintain that we need to keep track with a counter that is incremented as we request the next item.  This gives us pointer stability, but we also have the requirement that our pool must be able to grow.  

A solution using a dynamic array of our records can either provide growth or pointer stability, but not both.  

## Pointers Revisited 

In order to explain how we solve the dilemma of pointer stability and pool growth we need a brief review of pointers. 

When I think of pointers, I consider them as Native Unsigned Integers. These numeric values are memory locations. We already discussed that dynamic arrays are pointer types and that resizing an array simply copies memory over to an array of the new size. This means that addresses of all the elements in the array are modified*. The pointer value of a dynamic array is the address of the first element and therefore also changes. 

## Dynamic Arrays of Pointers 

With our understanding of pointers as numbers it should be clear that if we resize an array that holds pointers, then the new array will hold the same pointers. Remember, this is essentially the same as resizing an array of Native Unsigned Integers. The memory locations of the pointers change when we resize, but the values they hold remain the same. 

Below is an example of a memory block of a Dynamic with pointer value 1000 (64-bit).  

| Address | Item            | Value |
| ------: | ----            | ----: |
| 988     | Reference Count | 1     |
| 992     | Length          | 2     |
| 1000    | Pointer0        | 100   | 
| 1008    | Pointer1        | 200   |

 
Resizing the array to add an element moves the contiguous block of memory to a new location.   

| Address | Item            | Value |
| ------: | ----            | ----: |
| 1988    | Reference Count | 1     |
| 1992    | Length          | 2     |
| 2000    | Pointer0        | 100   | 
| 2008    | Pointer1        | 200   |
| 2016    | Pointer2        | 0     |
 

## Putting it all together 

Dynamic arrays are pointer types. While we can't resize the arrays that hold our records, we can safely resize an array that holds a dynamic array of records. We already concluded that we can’t resize these dynamic arrays of records, but we can add new ones. 

I will also use the term “row” to refer to the dynamic arrays that hold our records. Our pool can be defined as a generic type TPool<T> and the array that holds the arrays of our type is declared as TArray<TArray<T>>   

``` pascal
TPool<T> = Record 
private 
  Rows: TArray<TArray<T>>; 
... 

End; 
```

To access a pointer from the Pool we’ll add a method named New that returns a pointer. This aids in consumers understanding that it is similar in behavior to the loose function New. 

``` pascal
TPool<T> = Record 
private 
  Index: integer;  // initialized to 0 
  Rows: TArray<TArray<T>>; 

  Procedure AddARow; 
public 
  function New: Pointer; 
End; 
``` 
`New` can be implemented as follows 

 1. If we have no rows or the index is beyond the last index of our last row, then 
     1. add a row and 
     2. set the column to zero. 
2. Return the address of the element at index in the last row 
3. Increment the index 

``` pascal
function TPool<T>.New: Pointer;
begin
  if (Rows = nil) or (Index > High(Rows[High(Rows)])) then
     AddARow; // also reset Index to 0

  result := @Rows[High(Rows)][Index];

  inc(Index);
end;
```
 
We may need some flexibility in how to size new rows. For instance, we may not want to make every row the same length. We may grow the size of the rows as we allocate more rows as a way to reduce the number of allocations we make. For instance, each row could be made larger by a factor of the prior row, or we could decide the size of the row based on the overall size of all rows already allocated. In some cases we might have a very good estimation of the size and we don't want to aggressively grow our pool. Since the needs are determined by each use case, we could design our type to accept a method that would dictate the growth of the pool. 

In the solution below you can see one example of how we can implement our pool. I allocate rows of a fixed size and I don’t grow new rows at all. 


``` pascal
procedure TPool<T>.AddARow;
begin
  SetLength(Rows, Length(Rows) + 1);
  SetLength(Rows[High(Rows)], RowSize);

  Index := 0;
end;
```

We can declare a constant for RowSize for the number of entries, but we can get creative and allocate rows that fit a specific size in bytes. For instance, if we want 2KB rows we can declare our constant like this:

``` pascal
const
  RowSize = Trunc(2048/SizeOf(T)); 
```

Feel free to adjust this part of the code as needed. Consider potentially passing in a method for determining growth. In the complete code below you can see I also added an initializer, this is a feature that only exists in Delphi 10.4 and later. For older versions of Delphi, you can use a constructor that takes arguments, for instance you can set the initial row size or pass a growth function. 

``` pascal
unit Pool;

interface

type

TPool<T> = Record
private
const
  // We allocate rows of about 2KB each
  RowSize = Trunc(2048/SizeOf(T)); 
var
  Rows: TArray<TArray<T>>;
  Index: integer;

  procedure AddARow;
public
  function New: Pointer;

  // Record Initializers added in Delphi 10.4
  class operator Initialize (out Dest: TPool<T>);   
End;

implementation

procedure TPool<T>.AddARow;
begin
  SetLength(Rows, Length(Rows) + 1);

  // instead of a fixed RowSize consider growing rows 
  // as implementation demands
  SetLength(Rows[High(Rows)], RowSize); 

  Index := 0;
end;

class operator TPool<T>.Initialize(out Dest: TPool<T>);
const
  DefaultRec : TPool<T> = ();
begin
  Dest := DefaultRec;
end;

function TPool<T>.New: Pointer;
begin
  if (Rows = nil) or (Index > High(Rows[High(Rows)])) then
     AddARow;

  result := @Rows[High(Rows)][Index];

  inc(Index);
end;

end.
```

## Conclusion
I hope that you found this useful or at least interesting. Please download, clone or fork the [source code](https://github.com/schellingerhout/record-pool-delphi).

Comments and feedback are always welcomed