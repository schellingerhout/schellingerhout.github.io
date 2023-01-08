---
title: "An Array Builder for Delphi"
excerpt_separator: "<!--more-->"
categories:
  - Utilities
tags:
  - Delphi  
  - Arrays
  - Records
---

The design of an array builder in Delphi that holds records. Its purpose is to simplify the growth of an array as new elements are added, while improving speed by over-allocation

<!--more-->

## Motivation 
One of the common requirements while working with a dynamic array is the need to resize the array as new elements are added. We often have certain conditions that are checked before an element is added, and the initial size of the array may not be known.

Delphi does have classes that are wrappers around a dynamic array. The simple lists `TTList` and `TList<T>` are examples of these, but these are heap managed types. Dynamic arrays are superior to lists as a result from functions. If we return lists there is always the question of ownership and who is responsible for memory management. Even worse, caller does not even need to receive the return value.

## A Simple Example
We may have a trivial implementation such as this:

``` pascal
function PrimesBelow(MaxValue: integer): TArray<Integer>;
var
  Prime: integer;
begin
  Prime := 2;
  while Prime < MaxValue do
  begin
    SetLength(result, Length(Result) + 1);
    Result[High(result)] := Prime;
    Prime := NextPrime(Prime); // implementation not shown
  end;   
 end;
```
Not only is the code a bit ugly, it is quite inefficient to add one element at a time because dynamic array resizing involves copying the entire array. 

## Common attempts to improve performance

### Resize in Blocks

The first and most common attempt to improve is to allocate in chunks rather than one at time:

``` pascal
function PrimesBelow(MaxValue: integer): TArray<Integer>;
var
  Prime: integer;
  Count: integer;
const
  BlockSize: 100;
begin
  Count := 0;  
  Prime := 2;
  while Prime < MaxValue do
  begin
    if Count > High(Result) then
      SetLength(result, Length(Result) + BlockSize);
    
    Result[Count] := Prime;
    Inc(Count);
    Prime := NextPrime(Prime); // implementation not shown
  end;   

   SetLength(Result, Count); // we have to trim overallocation at the end
 end;
```
We also have to keep track of the actual account and resize the final result as well. We have made the code even uglier, plus we allocate in chunks of a fixed size. We may want to be a bit smarter in our sizing. 

### Smart Initial Allocation

In the case of prime numbers there are [functions to estimate the number of values](https://en.wikipedia.org/wiki/Prime-counting_function) that can be used in this case

``` pascal
function PrimesBelow(MaxValue: integer): TArray<Integer>;
var
  Prime: integer;
  Count: integer;
const
  BlockSize: 100;
begin
  Count := 0;  
  Prime := 2;

  SetLength(result, EstimatedNumberOfPrimesBelow(MaxValue)); // implementation not shown

  while Prime < MaxValue do
  begin
    // we might not be able to rely on our estimation
    // code and may need to check bounds anyway
    if Count > High(Result) then
      SetLength(result, Length(Result) + BlockSize);
    
    Result[Count] := Prime;
    Inc(Count);
    Prime := NextPrime(Prime); // implementation not shown
  end;   

  SetLength(Result, Count); 
end;
```
This code may be more performant, because of reducing the number of resizings of the array. However, the cost of the estimation function needs to be taken into account. Unfortunately, the code is still not very clean, in fact, uncertainty may lead us to still keep our bounds check and padding in place. We also probably would never have an exact size pre-calculated and we still need to fix the count at the end.

### Smart Array Growth

The other common practice is to grow the dynamic arrays using a function of the current size. In the case of `TList` we resize by 1.5 times the previous size (some small values have their own steps). We could define a growth function such as this

``` pascal
procedure GrowArray(var Array: TArray<Double>); 
begin
  if Array = nil then // an dynamic array in Delphi that has no elements is nil
    SetLength(Array, 4)
  else if Length(Array) < 5 then
    SetLength(Array, 8)
  else if Length(Array) < 9 then
    SetLength(Array, 16)  
  else
    SetLength(Array, (Length(Array) * 3) div 2;
end;
```
This function can be called instead of using the `SetLength(result, Length(Result) + BlockSize)` call we did earlier. For instance if our prime count estimation was too expensive vs. resizing arrays we could change the code as follows


``` pascal
function PrimesBelow(MaxValue: integer): TArray<Integer>;
var
  Prime: integer;
  Count: integer;

begin
  Count := 0;  
  Prime := 2;
  while Prime < MaxValue do
  begin
    if Count > High(Result) then
      GrowArray(result); 
    
    Result[Count] := Prime;
    Inc(Count);
    Prime := NextPrime(Prime); // implementation not shown
  end;   

  SetLength(Result, Count);
 end;
```
If we create an implementation we may want to pass the growth function in so that we can grow according to a strategic pattern. I briefly touched on this in [my previous post on record pools]({{ site.baseurl }}{% post_url 2022-12-10-record-pool-delphi %}). We could define a more flexible function such as this

``` pascal
function GrowthFunction(Size: integer): integer; 
begin
  if size = 0 then
    exit(4);
    
  if size < 5 then
    exit(8);

  if size < 9 then
    exit(16);

  result := (Size * 3) div 2)
end;
```
This function can then be passed grow the array as needed. I'll discuss this a bit later.

# TArrayBuilder<T> interface
The pattern above works quite well, but we can package all of this actions into a record that does all this for us. 

``` pascal
unit ArrayBuilder;

interface

Type

  TArrayBuilder<T> = Record
  private
    FData: TArray<T>;
    FCount: Integer;

    procedure Grow;
  public
    class function Init(Size: Integer): TArrayBuilder<T>; static;

    // Record Initializers added in Delphi 10.4
    class operator Initialize(out Dest: TArrayBuilder<T>);

    procedure Add(const Element: T);
    function GetArray: TArray<T>;
  end;

implementation

class operator TArrayBuilder<T>.Initialize(out Dest: TArrayBuilder<T>);
const
  DefaultRec: TArrayBuilder<T> = (); // count = 0, dynamic array empty
begin
  Dest := DefaultRec;
end;

class function TArrayBuilder<T>.Init(Size: Integer): TArrayBuilder<T>;
begin
  SetLength(Result.FData, Size);
end;

procedure TArrayBuilder<T>.Add(const Element: T);
begin
  if FCount > High(FData) then
    Grow;

  FData[FCount] := Element;
  Inc(FCount);
end;

procedure TArrayBuilder<T>.Grow;
var
  NewSize: Integer;
begin

  if FData = nil then
    NewSize := 4
  else if Length(FData) < 5 then
    NewSize := 8
  else if Length(FData) < 9 then
    NewSize := 16
  else
    NewSize := (Length(FData) * 3) div 2;

  SetLength(FData, NewSize);
end;

function TArrayBuilder<T>.GetArray: TArray<T>;
begin
  SetLength(FData, FCount);
  Result := FData;
end;

end.
```

Our code can now look like this:

``` pascal
function PrimesBelow(MaxValue: integer): TArray<Integer>;
var
  Prime: integer;
  PrimeBuilder: TArrayBuilder<Integer>;
begin
  Prime := 2;
  while Prime < MaxValue do
  begin
    PrimeBuilder.Add(Prime)  
    Prime := NextPrime(Prime); // implementation not shown
  end;   

  result := PrimeBuilder.GetArray;
end;
```

If we had a fast estimator for the number of primes we could initialize with that to improve performance, while keeping the code still relatively clean

```pascal
var
  Prime: integer;  
  PrimeBuilder: TArrayBuilder<Integer>;
begin
  PrimeBuilder := TArrayBuilder<Integer>.Init(
     EstimatedNumberOfPrimesBelow(MaxValue)); 
  Prime := 2;
  //... 
```
## Custom Growth Function

Our Array Builder can now be extended by allowing a growth function that can either be held by the `TArrayBuilder<T>` or passed into an overload of the `Add(Element: T)` method.

Here is an implementation using the function as a member of the `TArrayBuilder<T>`:

``` pascal
unit ArrayBuilder;

interface

Type

  TGrowthFunction = reference to function(CurrentSize: Integer): Integer;

  TArrayBuilder<T> = Record
  private
    FData: TArray<T>;
    FCount: Integer;
    FGrowthFunction: TGrowthFunction;
  public
    class function Init(Size: Integer): TArrayBuilder<T>; static;

    // Record Initializers added in Delphi 10.4
    class operator Initialize(out Dest: TArrayBuilder<T>);

    procedure Add(const Element: T);
    function GetArray: TArray<T>;
    procedure SetGrowthFunction(const F: TGrowthFunction);
  end;

  // due to the way Generics resolve I have to make the default growth function public
  function DefaultArrayBuilderGrowthFunction(Size: integer): integer;

implementation

function DefaultArrayBuilderGrowthFunction(Size: integer): integer;
begin
  if size = 0 then
    exit(4);

  if size < 5 then
    exit(8);

  if size < 9 then
    exit(16);

  result := (Size * 3) div 2;
end;


class operator TArrayBuilder<T>.Initialize(out Dest: TArrayBuilder<T>);
const
  DefaultRec: TArrayBuilder<T> = (); // count = 0, dynamic array empty
begin
  Dest := DefaultRec;
  Dest.FGrowthFunction := DefaultArrayBuilderGrowthFunction;
end;

procedure TArrayBuilder<T>.SetGrowthFunction(const F: TGrowthFunction);
begin
  FGrowthFunction := F;
end;

class function TArrayBuilder<T>.Init(Size: Integer): TArrayBuilder<T>;
begin
  SetLength(Result.FData, Size);
end;

procedure TArrayBuilder<T>.Add(const Element: T);
begin
  if FCount > High(FData) then
     SetLength(FData, FGrowthFunction(Length(FData)));

  FData[FCount] := Element;
  Inc(FCount);
end;

function TArrayBuilder<T>.GetArray: TArray<T>;
begin
  SetLength(FData, FCount);
  Result := FData;
end;

end.
```

We can now provide a custom growth function in this form

``` pascal
SomeArrayBuilder.SetGrowthFunction(
  function (Size: integer): integer 
  begin
    result := Size + 100;
  end
);
```


## Conclusion
I hope that you found this useful or at least interesting. Please download, clone or fork the [source code](https://github.com/schellingerhout/array-builder-delphi).

Comments and feedback are always welcomed