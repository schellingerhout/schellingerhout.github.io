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

## Quick Recap ##
### Records can interpret data in different ways ###
As mentioned in prior sections that pointers are simply numbers that correspond to places in memory and that types associated with that address are the way we interpret the data. In many cases data can be interpreted as compatible data types. Here is an example of we could define a customer record at a transmitter:


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

### Someone needs to manage the memory ###

In the scenario described above the sender holds the record that has the `String` type. This type just like dynamic arrays (like `TArray<Double>`) are reference counted types. Referencing the underlying data of these types as either `PChar` for `String`, or `PDouble` for `TArray<Double>` does not increment the reference count. This means that as soon as the dynamic array or string is out of scope the memory is disposed and the pointer is invalid.

*Note:* Dynamic arrays and strings must be kept alive via a reference as long as there is a pointer to their underlying data.
{: .notice}

There are a few ways to manage this. 
* Receiving party allocates memory for the transmitting party to fill with data. The receiving party disposes the memory after it processed it (and at its convenience).
* Transmitting party allocates memory for the receiving party to copy or process completely (no references held). The transmitting party disposes the memory 
* once the send method returns, or 
  * waits to be notified that the receiving party is done with the data, or
  * the receiving party terminates the session

The scenario of allocating memory to be filled is very common in the Windows API, this is often a two-step call. The first call would pass the buffer as `nil` and receive the size that you need to allocate.

In the rest of this discussion I will focus on the easier method of synchronous transmission where I assume data is received and copied or completely processed into other types at the receiver before I return. This way the transmitter can dispose as soon as the method call into the receiver returns

### Abstract types can be used to direct data ###

In [Section 1]({{ site.baseurl }}{% post_url 2019-02-11-datatransmission1 %}) I covered the idea of abstract structured types. The idea was that if we had a record that shared part of its structure with another, we could safely interpret a pointer (memory address) as the smaller abstract of these types (sometimes called a header record). This is the same concept used in Windows messaging where `DEV_BROADCAST_HDR` is essentially the header part of other records such as `DEV_BROADCAST_DEVICEINTERFACE` or `DEV_BROADCAST_VOLUME`. The determination of how the memory could be fully interpreted was contained in `dbcv_devicetype`.

{% highlight pascal %}
...
  case LPDeviceBroadcastHeader^.dbch_devicetype of
      DBT_DEVTYP_DEVICEINTERFACE:
        begin 
          LPBroadcastDeviceIntf := PDEV_BROADCAST_DEVICEINTERFACE(LPDeviceBroadcastHeader);
          LUsbDeviceName := PChar(@LPBroadcastDeviceIntf^.dbcc_name);
      ...
        end;

      DBT_DEVTYP_VOLUME:
        begin
          LPBroadcastVolume := PDEV_BROADCAST_VOLUME(LPDeviceBroadcastHeader);
          // use LPBroadcastVolume.dbcv_unitmask to find volume information
          ...
        end; 
    end; 
...    
{% endhighlight %}  

Our receiver can direct data in the same way:
* read part of the data, 
* use the partial data to determine the type and size of memory to read
* read the data as defined by the underlying type

## Building our API ##

I will create a sample data environment containing Lines, Arcs, PolyLines and GeometryLists. Sender and Receiver records will be able to interpret data in compatible ways, with the Sender responsible for data lifetime

### The Sender's data ###
{% highlight pascal %}

type
{$Z4} //integer enum
  TxRectTypeEnum = (TxRectType_Point,  TxRectType_Line, TxRectType_Arc, TxRectType_Polyline, TxRectType_GeometryList);
{$Z1}
const
  TxRectType_Undefined = TxRectTypeEnum(-1);

Type
  //The base memory block of all parameter recs, also serves as a signal parameter (no data transmitted)
  PTxRec = ^TxRec;
  TxRec = Record
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum; // Integer
  End;

  PointRec = Record   //not transmitted directly, composed type
    X, Y : double;
  End;

  PTxPointRec = ^TxPointRec;
  TxPointRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum;

    //Point Specific
    p: PointRec;
  End;

  PTxLineRec = ^TxLineRec;
  TxLineRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum;

    //Line Specific
    p1, p2 : PointRec;
  End;

  PTxArcRec = ^TxArcRec;
  TxArcRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum;

    // Arc Specific
    p : PointRec;
    CCW: Boolean;   //in Delphi a Boolean has size of Byte
    StartAngle, EndAngle: Double;
  End;


  PTxPolyLineRec = ^TxPolyLineRec;
  TxPolyLineRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum;


    // Polyline Specific
    VertexCount: Uint32;
    Vertices : TArray<PointRec>;    // since x can have a valid value of zero, we need a count, we can't use null termination
  End;

  PTxGeometryListRec = ^TxGeometryListRec;
  TxGeometryListRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: TxRectTypeEnum;


    // PolyLineArcRec Specific
    Geometry : TArray<PTxRec>;   // since we have an array of pointers, this array can be null terminated
  End;
{% endhighlight %}  

### The Receiver's data ###
Dynamic arrrays are mapped to pointer types.

{% highlight pascal %}
type
{$Z4} //integer enum
  RxRectTypeEnum = (RxRectType_Point,  RxRectType_Line, RxRectType_Arc, RxRectType_Polyline, RxRectType_GeometryList);
{$Z1}
const
  RxRectType_Undefined = RxRectTypeEnum(-1);
Type

//The base memory block of all parameter recs, also serves as a signal parameter (no data transmitted)

PPRxRec = ^PRxRec; //array of pointers to  RxRec
  PRxRec = ^RxRec;
  RxRec = Record
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum; // Integer
  End;

  PPointRec = ^PointRec;
  PointRec = Record
    X, Y : double;
  End;

  PRxPointRec = ^RxPointRec;
  RxPointRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum;

    //Point Specific
    p: PointRec;
  End;

  PRxLineRec = ^RxLineRec;
  RxLineRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum;

    //Line Specific
    p1, p2 : PointRec;
  End;

  PRxArcRec = ^RxArcRec;
  RxArcRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum;

    // Arc Specific
    p : PointRec;
    CCW: Boolean;   //in Delphi a Boolean has size of Byte
    StartAngle, EndAngle: Double;
  End;

  PRxPolyLineRec = ^RxPolyLineRec;
  RxPolyLineRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum;

    // Polyline Specific
    VertexCount: Uint32;
    Vertices : PPointRec;  
  End;

  PRxGeometryListRec = ^RxGeometryListRec;
  RxGeometryListRec = Record
    // Common header
    Size: Cardinal; //UInt32
    RecType: RxRectTypeEnum;

    // GeometryListRec Specific
    Geometry : PPRxRec;   //  null terminated
  End;
{% endhighlight %}
  
### Receiver's DLL Export ###
 
The Receiver can export a method of this format:
  
{% highlight pascal %}
procedure SendTxRecord(APRxRec : PRxRec); stdcall;
begin
  case APRXRec.RecType of
    RxRectType_Point :
      ReceivePoint(PRxPointRec(APRxRec));
    RxRectType_Line : 
      ReceiveLine(PRxLineRec(APRxRec));
    RxRectType_Arc : 
      ReceiveArc(PRxArcRec(APRxRec));
    RxRectType_Polyline :
      ReceivePolline(PRxPolyLineRec(APRxRec));
    RxRectType_GeometryList :
     ReceivePolline(RxGeometryListRec(APRxRec));
  end;
end;  

exports SendTxRecord;
{% endhighlight %}
  
It seems strange that the sender would have a method called "Send", but the receiver will not use it internally, instead a the Sender can link to it as 

{% highlight pascal %}
  procedure SendTxRecord(APTxRec : PTxRec); stdcall; external 'MyDLL.dll'
{% endhighlight %}

We can also add a similar method to process arrays of data that can be traversed with pointermath as discussed in Section 2. 

{% highlight pascal %}
procedure SendTxRecords(APRxRec : PRxRec; ACount: integer); stdcall;
begin
  case APRXRec.RecType of
    RxRectType_Point :
      ReceivePoints(PRxPointRec(APRxRec), ACount);
    RxRectType_Line : 
      ReceiveLines(PRxLineRec(APRxRec), ACount);
    RxRectType_Arc : 
      ReceiveArcs(PRxArcRec(APRxRec), ACount);
    RxRectType_Polyline :
      ReceivePollines(PRxPolyLineRec(APRxRec), ACount);
    RxRectType_GeometryList :
      ReceivePollines(RxGeometryListRec(APRxRec), ACount);
  end;
end;  
{% endhighlight %}



### Preparing Data for Transmission ###
Each of our datatypes will be ready to receive the relevant information to transmit, but the values that are needed for processing on the receiver side such as `size` and `rectype` will need to be populated for every record before we can fill it with the data that we want to transmit. 

Delphi does not have a way to auto-initialize records. Only the smart pointer types: Strings and Interfaces are automatically initialized as null. [You could exploit the use of properties along with a string or interface field on the record to initialize it on demand](https://stackoverflow.com/questions/39392920/how-can-delphi-records-be-initialized-automatically), but that will not help us in the transmission of this record //put link to stackoverflow. We are passing an abstract type and there are no virtual calls on record structures. Even if these did exist, support would vary by programming language. Also, we really want our records to be a data map of memory and nothing more.

Delphi provides for a simple syntax to declare record constants, for instance:

{% highlight pascal %}
const 
  DefaultTxArcRec: TTxArcRec = ();
{% endhighlight %}
  
This constant as defined would be a default record with all memory zeroed out. This means integer and floating numeric values are zero and booleans are false. Zeroed out memory also means pointers are nil, which in turn means dynamic arrays are empty and strings are empty strings.

We can do even better with our constant, when we declare fields and values using `field: value` syntax then only the declared fields receive values other than 0 all other values are still blank. Here is the improved constant:


{% highlight pascal %}
const 
  DefaultTxArcRec: TTxArcRec =
    (     
      Size: SizeOf(TTxArcRec);
      RecType: TxRectType_Arc;      
     );
{% endhighlight %}
  
Constant record declarations can also be nested with the same considerations.

So transmission process would be :
* Obtain a record filled with appropriate `size` and `rectype` values
* Populate the data to transmit
* Transmit the record
* If the record was dynamically allocated then dispose it after transmission
  
### Generalizing transmission ###
The process of transmission as described above follows a predictable path and may lend itself well to using generics and anonymous methods. The problem is that pointers to records do not give us virtual type info, but we could use the typeinfo of a specific record type to generalize our data.

Here is the basic idea: When sending a record call a generic method that gets specialized by the record type `Send<TxLineRec>(...)`,  we could also pass an anonymous callback method so we can be populate the data in the record (beside the header). This anonymous method will give the caller the record to manipulate via a var parameter.  If we transmit a list we'll have to pass a count and receive an anonymous callback that presents both the record and its index in the list.

To facilitate the generic calls, I'll wrap these `Send` calls in a class

## The Transmitter Class ##

### Interface ###
Here is an example of what my transmitter class could look like:

{% highlight pascal %}
Type

  TSendConfigProc<T> = reference to procedure(var A: T);
  TSendConfigProcIter<T> = reference to procedure(var A: T; AIndex: integer);

  TTxer = class
    public

     class procedure SendRecord(ARecord: PTxRec); static;
     class procedure SendRecords(ARecordArray: PTxRec; ACount: integer); static;

     class procedure Send<T>(AConfigureProc: TSendConfigProc<T>); overload;
     class procedure Send<T>(ANumRecords: integer; AConfigureProc: TSendConfigProcIter<T>); overload;
   end;
{% endhighlight %}

For our transmitter class we'll add the ability to send records and an array of records with a `Send` command that is generic and will take either an anonymous method to configure a record, or a count and an anonymous method that will configure a record, plus provide the index of the active item.

The sample transmitter class also has `SendRecord` and `SendRecords` that are direct one-to-one wrappers of the DLL signatures. I'll explain the reason for these a bit later.

### Generic Methods ###

{% highlight pascal %}

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
  LDefault : T;
begin
  SetLength(LDynArray, ANumRecords);

  LDefault := TxRec.Default<T>;

  for i := 0 to ANumRecords-1 do
  begin
    LDynArray[i] := LDefault;
    AConfigureProc(LDynArray[i], i);
  end;
  SendRecords(@LDynArray[0], ANumRecords);
end;

{% endhighlight %}

These calls facilitate the steps in our process as follows
* We obtain a record filled with appropriate `size` and `rectype` values by our `TxRec.Default<T>` method
* The API Consumer populate the data to transmit via the anonymous methods of types `TSendConfigProc<T>` and `TSendConfigProcIter<T>`
* The record is transmitted via `SendRecord` and `SendRecords`
* The records are disposed when the `L: T` variable and the reference counted `TArray<T>` dynamic array going out of scope

We will examine each of these in more detail

### Initializing Records ###

Delphi constants allow for a loophole to modify values that should not change, all you have to do is obtain a pointer to the constant and you can manipulate the values. This makes using constants with pointer types particularly dangerous. 

In this code you will see that I keep the scope of referencing the constant via pointer as small as possible. 
{% highlight pascal %}

class function TxRec.Default<T>: T;
var
  PT: ^T; // this will be a pointer to a const, do not modify values via this pointer
begin
  
  if TypeInfo(T) =  TypeInfo(TxPointRec) then
    PT := @DefaultPointRec
  else if TypeInfo(T) =  TypeInfo(TxLineRec) then
    PT := @DefaultLineRec
  else if TypeInfo(T) =  TypeInfo(TxArcRec) then
    PT := @DefaultArcRec
  else if TypeInfo(T) =  TypeInfo(TxPolyLineRec) then
    PT := @DefaultPolyLineRec
  else if TypeInfo(T) =  TypeInfo(TxGeometryListRec) then
    PT := @DefaultGeometryListRec
  else
    PT := nil; // raise exception

  result := PT^; //We Copy value, so the constant is not inadvertently modified
end;

{% endhighlight %}

You may recall that I added methods `SendRecord` and `SendRecords` to my transmitter class, the reason I did so was that a generic method cannot use the imported DLL method directly unless it is in the interface section of the unit. Similarly here: The defaultrecords will need to be declared in the interface section with the records. Unfortunately, even though records support constant declaration within the record name scope, we cannot declare a self constant such as we can with classes. So we'll have to declare these record constants separate from our records in the interface section of the unit.

### Populating the Record ###

The anonymous method passed by the consumer of the API serves as a way to populate the record

{% highlight pascal %}
  TXer.Send<TxLineRec>( 
    procedure(var ARec: TxLineRec) 
    begin
      ARec.p1.x := 0.5;
      ARec.p2.y := 0.25;
      ARec.p2.x := 1.0;
      ARec.p2.y := 2.0;
    end
  );
  
  Txer.Send<TxPolLineRec>(FPollines.Count, 
  Procedure(var ARec: TxPolLineRec; AIdx: integer)
  begin
    ARec.VertexCount := Length(FPollines[AIdx].Vertices);
    ARec.Vertices := FPollines[AIdx].Vertices;  
  end
  );
{% endhighlight %}

Smart reference types such as strings, interfaces and dynamic arrays that are cast as dumb pointer types must be kept alive for the duration of transmission. In the case above assume `Vertices` on our polline objects in our list is a `TArray<PointRec>` and we have one on our record so the reference will be kept alive. If we had a field on our record of type `PPointRec` then we should ensure that the list element's array does not get modified.

### Transmitting the Record ###


The pointer to the record or array of records are handed off to the dll which should process the data synchronously (at least with the method that creates, transmits and disposes the data). We could add our DLL imports to the interface section and call them directly, or keep them in the implementation section and call them via the class. The class methods will just pass the parameters along

{% highlight pascal %}
class procedure TTxer.SendRecord(ARecord: PTxRec);
begin
   SendTxRecord(ARecord); //dll call
end;

class procedure TTxer.SendRecords(ARecordArray: PTxRec; ACount: integer);
begin
  SendTxRecords(ARecordArray, ACount); //dll call
end;
{% endhighlight %}

### Disposal of the Record ###

Disposal is done via reference counting of the dynamic array in the case of an array transmission, and in the case of a single record it will be disposed once it exists the scope of the `Send` method. We could also allocate a record via `New`. In that case we'd do something like this

{% highlight pascal %}
class procedure TTxer.Send<T>(AConfigureProc: TSendConfigProc<T>);
Var
  LPT: ^T;
begin
  New(LPT);
  try
    LPT^ := TxRec.Default<T>;
    AConfigureProc(LPT^);
    SendRecord(PTxRec(LPT));
  finally
   Dispose(LPT);
  end;
end;
{% endhighlight %}

I personal prefer the stack cleaning up the variable, but there may be cases where this method may be justified.

## Conclusion ##

The repository for this blog series code can be found [here](https://github.com/schellingerhout/data-transmission-delphi).

That concludes this series on data transmission with records and arrays. Hopefully you'll be able to extend these concepts to other programming languages. C\C++ should be easy candidates for handling data transmission in this way, for .Net you'd probably have to write a wrapper class in C++\CLI because it may be tricky to write the proper PInvoke headers to process data.

