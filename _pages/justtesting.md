---
title: "Data Transmission with Delphi Redux"
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
sidebar:
  nav: data_transmission
sitemap: false
permalink: /justtesting.html
---

In this blog post I will revisit my series from four years ago and update it with more current technology

<!--more-->
Revisiting the series that deal with the transfer of raw data at high speed with direct access using pointers to structures that can be supported by most programming platforms. 

## Why am I revisiting a four year old post? ##
I was never happy with the solution to get default values based on the generic type and new functionality in Delphi has allowed me to clean up some of the code I wrote for [Part 3]({{ site.baseurl }}{% post_url 2019-09-12-datatransmission3 %}) of my series on data transmission. 

I want to share what I found, how it can clean up the code from the prior article and potential for other uses in your own projects.

### In case you missed it ###
I finished my original series of three articles exactly four years ago to the date. If you missed it here are some links to the original posts.
* [Part 1: Pointers and Structures]({{ site.baseurl }}{% post_url 2019-02-11-datatransmission1 %})
* [Part 2: Arrays and Pointer Arithmetic]({{ site.baseurl }}{% post_url 2019-02-14-datatransmission2 %})
* [Part 3: Transmitting and Interpreting Data]({{ site.baseurl }}{% post_url 2019-09-12-datatransmission3 %})

Here is the gist for those that don't want to read that much:
If two parties share a pointer and some common understanding of the structure of the memory at that location, then we can have very fast communication. Since arrays are contiguous blocks of memory we can rapidly advance through memory and read structures provided that we know the size of each of the structures, their composition and the number of structures we need to read.

The end result was a library example using closures to configure records for transmission. In my example I created a transmitter `TTxer` that can send any of a number of geometry entity records defined using generics.


``` pascal
//Transmit records one by one
 TTXer.Send<TxLineRec>( 
    procedure(var R: TxLineRec) 
    begin
      R.p1.x := 0.5;
      R.p1.y := 0.25;
      R.p2.x := 1.0;
      R.p2.y := 2.0;
    end
  );
  
 // Transmit records as an array (pointer and count)
  TTxer.Send<TxPolyLineRec>(FPolylines.Count, 
    Procedure(var R: TxPolyLineRec; i: integer)
    begin
      R.VertexCount := Length(FPolylines[i].Vertices);
      R.Vertices := FPolylines[i].Vertices;  
    end
  );
```  

## Getting defaults from generics types are messy ##
In order to transmit my data I had to initialize each of my records before passing them to the anonymous callback closure. Below is the code to send individual and arrays of records. It is rather elegant:

``` pascal
class procedure TTxer.Send<T>(AConfigureProc: TSendConfigProc<T>);
Var
  L: T;
begin
  L := TxRec.Default<T>;
  AConfigureProc(L);
  SendRecord(@L);
end;

class procedure TTxer.Send<T>(ANumRecords: integer; AConfigureProc: TSendConfigProcIter<T>);
Var
  LDynArray: TArray<T>;
  i: integer;
  LDefault: T;
begin
  SetLength(LDynArray, ANumRecords);

  LDefault := TxRec.Default<T>;

  for i := 0 to ANumRecords - 1 do
  begin
    LDynArray[i] := LDefault;
    AConfigureProc(LDynArray[i], i);
  end;
  SendRecords(@LDynArray[0], ANumRecords);

end;
```

However, the nastiness was factored out and hidden in `TxRec.Default<T>`, which had to figure out type information for type T and then return a default for it. Since generics are generated for each type the code was duplicated for each type that filled `T` for `class function TxRec.Default<T>: T;`. 

``` pascal
class function TxRec.Default<T>: T;
var
  PT: ^T; // this will be a pointer to a const, do not modify values via this pointer
begin

  if TypeInfo(T) = TypeInfo(TxPointRec) then
    PT := @DefaultPointRec
  else if TypeInfo(T) = TypeInfo(TxLineRec) then
    PT := @DefaultLineRec
  else if TypeInfo(T) = TypeInfo(TxArcRec) then
    PT := @DefaultArcRec
  else if TypeInfo(T) = TypeInfo(TxPolyLineRec) then
    PT := @DefaultPolyLineRec
  else if TypeInfo(T) = TypeInfo(TxGeometryListRec) then
    PT := @DefaultGeometryListRec
  else
    PT := nil; // raise exception

  result := PT^; // We Copy value, so the constant is not inadvertently modified
end;
```

I really dislike this code and tried to find alternatives. I investigated the build in function `Default(T)` that is used in the Generic Collections to set or return default values, but I could not find a way to override its behavor for my types. The code for `Default(T)` is not in system.pas and may be some compiler magic. I finally resolved to leave it as such.

### Delphi 10.4 has some new tricks ###
Unbeknownst to most Delphi developers two new operators were snuck into Delphi 10.4, they are not even in the installed help file, but only mentioned in the online the documentation and in a blog post by Marco Cantu. These two operators are `Initialize` and `Finalize` and the implications of these are huge (more on this later).

`Initialize` allows us to define code that runs when a record first enters into scope, plus and most astounding to me, the code for `Initialize` is also called per element in an array if its allocated. 

To make our record a "managed" record we add these two operators 

``` pascal
TMyRecord = record

  // record data not relevant to illustration

  class operator Initialize (out Dest: TMyRecord);
  class operator Finalize(var Dest: TMyRecord);
end;
```

### Replacing `TxRec.Default<T>` with `Intialize` operator ###
In our case we don't box any types that need to be disposed so we don't need `Finalize`, but we can use `Initialize` per each of our records to have them load their size and enumerated type.

For instance, our line record can be changed to look like this:
{% highlight pascal %}

TxLineRec = Record
  // Common header
  Size: Cardinal; // UInt32
  RecType: TxRectTypeEnum;

  // Line Specific
  p1, p2: PointRec;

  class operator Initialize(out Dest: TxLineRec);
End;
...

class operator TxLineRec.Initialize(out Dest: TxLineRec);
begin
  Dest.Size := SizeOf(TxLineRec);
  Dest.RecType := TxRectType_Line;
  // p1 and p2 are not initialized
end;
```

The rest of the records can similarly be adjusted. This means we no longer have one central function that checks the type information to determine the default value to return, but rather each type controls their own initialization.

Our send code is now even cleaner

``` pascal
class procedure TTxer.Send<T>(AConfigureProc: TSendConfigProc<T>);
Var
  L: T;
begin
  AConfigureProc(L);
  SendRecord(@L);
end;

class procedure TTxer.Send<T>(ANumRecords: integer; AConfigureProc: TSendConfigProcIter<T>);
Var
  LDynArray: TArray<T>;
  i: integer;
  LDefault: T;
begin
  SetLength(LDynArray, ANumRecords);

  for i := 0 to ANumRecords - 1 do
    AConfigureProc(LDynArray[i], i);

  SendRecords(@LDynArray[0], ANumRecords);
end;
```  

## Cost and Benefits of Managed records ##
When we call SetLength and new records are allocated for the array there is one call to `Initialize` per element. In my older code it was a faster local memory copy in the loop before configuring the record before transmission.

I don't know what other overheads are associated with managed records. They seem to be more analogous to C++ classes and structs that exist on the stack instead of dynamically allocated on the free store (like all Delphi objects). Testing would be needed to see if there is any extra processing. In my basic testing I did not notice any differences.

Managed records allow us to have stack managed types that can box types that are dynamically allocated on the free store (colloquially called the heap). Dynamically allocated types can be constructed in `Initialize` and disposed in `Finalize` without developers needing to call constructors and box code in try-finally blocks. These types are scope managed.

## Other Improvements to the Transmitter Class  ##
I realized in rewriting some of the code that signaling that does not require any configuration, or records that can be transmitted once configured based on their own `Initialize` code. So there are now three overloads for `Send<T>`.

I also considered the need for end-users to control records themselves without a configuration callbacks, and to facilicate that, I added two overloads for `SendRecords`

``` pascal
TTxer = class
private

public

  class procedure SendRecord(ARecord: PTxRec); static;
  class procedure SendRecords(ARecordArray: PTxRec; ACount: integer); static;

  class procedure Send<T>(); overload;  // signal type, no configured data
  class procedure Send<T>(AConfigureProc: TSendConfigProc<T>); overload;
  class procedure Send<T>(ANumRecords: integer; AConfigureProc: TSendConfigProcIter<T>); overload;
end;
```


## Conclusion ##

The updated repository for this blog series code can be found [here](https://github.com/schellingerhout/data-transmission-delphi/tree/Delphi10.4). You will notice that its a branch named Delphi10.4, the original source code is still under the master branch
