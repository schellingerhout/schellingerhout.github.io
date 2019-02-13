---
title: "Data Transmission (Part 1: Common knowledge)"
excerpt_separator: "<!--more-->"
categories:
  - DLL
  - Records
  - Structs
  - Pointer Math
tags:
  - Delphi  
  - Communication
  - Cross-Platform
---

In this blog post I will start to describe the data structures passed in communicating cross-platform via DLLs. 

<!--more-->
The purpose of this blog series is to understand how to transfer raw data at high speed with direct access using pointers to structures that can be supported by most programming platforms.

When we transmit data between an application and a DLL we operate in the same memory space. This means that we can read memory directly via pointers. This allows high speed transmission of data by directly reading memory as common structures shared between the application and DLLs. 

## Pointer Primer ##

Many tutorials on pointers exist already and this is not a replacement for those, just a basic primer and refresher on high level concepts. 

Think of memory as a grid of boxes where each box has a sequential identifier (or address). If we have a memory position in this grid as well as knowledge about the number of boxes to read beyond that position, and their interpretation, then two systems with this common understanding can create an efficient system of directly sharing information.

In the model described above a pointer is simply the address of a box as described above. A pointer can simply be seen as a simple numeric value like an integer. I like to think of pointers as "native unsigned integers". Thinking about pointers this way helps keep our sanity when passing pointers by value or by reference. The value held by the pointer behaves in the same way as a numeric value passed by value or by reference. 

Let us look at a sample of a pointer type defined in System.SysUtils.pas

{% highlight pascal %}
PThreadInfo = ^TThreadInfo;
TThreadInfo = record
  Next: PThreadInfo;
  ThreadID: TThreadID;
  Active: Integer;
  RecursionCount: Cardinal;
end;
{% endhighlight %}

`PThreadInfo` is a pointer type to `TThreadInfo`. The type `PThreadInfo` has exacly the same size as `Pointer`, which is the same size as `NativeUInt` (32-bit or 64-bit based on the platform). So if the `PThreadInfo` essentially just holds a numeric value just like any other pointer, then why declare it in this way? Well, as I mentioned earlier, if we know how much memory we need to read and how to interpret its values then we know what the data represents. When we de-reference a typed pointer variable like PThreadInfo (calling in this form: `LMyThread^`), then we interpret the memory at the position at the value held by `PThreadInfo` as defined by `TThreadInfo`.

Besides an understanding of the data at the memory location of the pointer, the strongly typed pointer also has benefits in an area called "pointer math". In short, this means that if we increment our pointer it will not always increment by 1, but by the size of the associated type. Pointer math also allows for indexing memory positions based on a pointer type. I will explain in more detail in a future post. 

Now we have both parts to make our communication between application and DLL possible: a memory address and an interpretation of that memory. Let us look at an example of what our communication might look like

## Abstract Pointer Types in Window Messaging ##

Windows messaging deals with lots of varying information and types in a generic way. Let us look at one scenario of windows messaging how this works with abstract treatment of  pointers. 

The windows message [WM_DEVICECHANGE](https://docs.microsoft.com/en-us/windows/desktop/DevIO/wm-devicechange) can inform us of device changes on our system. Once we receive this method and the `wmparam` has a value of [DBT_DEVICEARRIVAL](https://docs.microsoft.com/en-us/windows/desktop/DevIO/dbt-devicearrival) we know that a device arrived. Then we can read the `lparam` for information for device. In the windows message strucutre `lparam` is simply a `NativeInt` (i.e. a numeric value), and for this windows message we can treat it as a pointer to [DEV_BROADCAST_HDR](https://docs.microsoft.com/en-us/windows/desktop/api/Dbt/ns-dbt-_dev_broadcast_hdr)

The pointer and the structure it refers to looks like this in Delphi:

{% highlight pascal %}
PDEV_BROADCAST_HDR = ^DEV_BROADCAST_HDR;
DEV_BROADCAST_HDR = record
  dbch_size: DWORD;
  dbch_devicetype: DWORD;
  dbch_reserved: DWORD;
end;
{% endhighlight %}

The information we can glean form this is limited to essentially just the device type, so how can we get more detail? If you read the documentation for [DEV_BROADCAST_HDR](https://docs.microsoft.com/en-us/windows/desktop/api/Dbt/ns-dbt-_dev_broadcast_hdr). You will see that for each of the values of the device type it tells you that the structure is not really `DEV_BROADCAST_HDR`, but some other structure like [DEV_BROADCAST_DEVICEINTERFACE](https://docs.microsoft.com/en-us/windows/desktop/api/dbt/ns-dbt-_dev_broadcast_deviceinterface_a) or [DEV_BROADCAST_VOLUME](https://docs.microsoft.com/en-us/windows/desktop/api/dbt/ns-dbt-_dev_broadcast_volume). You may be confused at this point, but let us look at those two types as translated to Delphi code:


{% highlight pascal %}
PDEV_BROADCAST_DEVICEINTERFACE = ^DEV_BROADCAST_DEVICEINTERFACE;
DEV_BROADCAST_DEVICEINTERFACE = record
  dbcc_size: DWORD;
  dbcc_devicetype: DWORD; // = DBT_DEVTYP_DEVICEINTERFACE
  dbcc_reserved: DWORD;
  dbcc_classguid: TGUID;
  dbcc_name: Char;
end;

PDEV_BROADCAST_VOLUME = ^DEV_BROADCAST_VOLUME;
DEV_BROADCAST_VOLUME = record
  dbcv_size: DWORD;
  dbcv_devicetype: DWORD; // = DBT_DEVTYP_VOLUME
  dbcv_reserved: DWORD;
  dbcv_unitmask: DWORD;
  dbcv_flags: Word;
end;
{% endhighlight %}  

Look at the two records above and compare them to [DEV_BROADCAST_HDR](https://docs.microsoft.com/en-us/windows/desktop/api/Dbt/ns-dbt-_dev_broadcast_hdr). You will notice that the first part of their data is the same. Once we cast our pointer from the header type (`PDEV_BROADCAST_HDR`) to the specific type (say `PDEV_BROADCAST_DEVICEINTERFACE`) we can now read more information. All of this works of course if both parties have a clear and unambiguous understanding of the size of data that needs to be read.

Without going into detail. Here is an example of how we would use this

{% highlight pascal %}
procedure TFoo.WMDeviceChange(var AMessage: TMessage);
var
  LUsbDeviceName: string;
  LPDeviceBroadcastHeader: PDEV_BROADCAST_HDR;
  LPBroadcastDeviceIntf: PDEV_BROADCAST_DEVICEINTERFACE;
  LPBroadcastVolume: PDEV_BROADCAST_VOLUME;
begin
  if (AMessage.wParam = DBT_DEVICEARRIVAL) then
  begin
    LPDeviceBroadcastHeader := PDEV_BROADCAST_HDR(AMessage.LParam);

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
  end;
end;
{% endhighlight %}  

## Shared Structures (Naive Attempt) ##

As long as our application and DLL have a common understanding of the structure they need to pass the call could be a simple one

{% highlight pascal %}
procedure foo(AData: Pointer); stdcall;
{% endhighlight %}  

The DLL receiving the information can cast to the correct pointer type and read the memory at that address, but this would not be very extensible and reduce readability of the source code. Look at the windows message handler routine we defined earlier. I that case we use the less abstract `PDEV_BROADCAST_HDR` type and cast to the appropriate `PDEV_BROADCAST_*` pointer type based on `dbch_devicetype`. We can do the same in our shared structure (record) definitions.


Let use start by creating our header or "base" record type. As seen above we need a type so we can cast to correct pointer type. An enumeration would work fine

{% highlight pascal %}
type
  TxRectTypeEnum = (TxRectType_Undefined, TxRectType_Line, TxRectType_Arc);
  
  PTxRec = ^TxRec;
  TxRec = Record
    RecType: TxRectTypeEnum;
  End;
{% endhighlight %}  


Next we can define our Line and Arc records, but we have to keep in mind that the first data needs to match that of TxRec.

{% highlight pascal %}
type

  PTxRecArc = ^TxRecArc;
    TxRecArc = Record
    // common to TxRec   
      RecType: TxRectTypeEnum; // = TxRectType_Arc
    
    // TxRecArc data start    
    CenterX: Double;
    CenterY: Double;
    StartAng: Double; 
    EndAng: Double;
    Radius: Double;
    
    CCW: boolean    
  End;
  
  PTxRecLine = ^TxRecLine;
  TxRecLine = Record
    // common to TxRec     
      RecType: TxRectTypeEnum; // = TxRectType_Line
    
    // TxRecLine data start    
    P1X: Double;
    P1Y: Double;
    
    P2X: Double;
    P2Y: Double;
  End;  
  
{% endhighlight %}  

The type alone will work for single records transmitted by our API. We will need the size if we read a complex stream of data (for instance from a file). It would be prudent to also add the size to our base type. If we ever encounter a stream that has a structure we don't understand we can skip that section of memory. 

## Shared Structures (Slightly Improved) ##

{% highlight pascal %}
type
  PTxRec = ^TxRec;
  TxRec = Record
    Size: DWord;  
    RecType: TxRectTypeEnum;
  End;
  
  PTxRecArc = ^TxRecArc;
  TxRecArc = Record
    // common to TxRec  
    Size: DWord;
    RecType: TxRectTypeEnum; // = TxRectType_Arc

    // TxRecArc data start    
    CenterX: Double;
    CenterY: Double;
    StartAng: Double; 
    EndAng: Double;
    Radius: Double;
    
    CCW: boolean    
  End;
  
  PTxRecLine = ^TxRecLine;
  TxRecLine = Record
    // common to TxRec 
    Size: DWord;  
    RecType: TxRectTypeEnum; // = TxRectType_Line
  
    // TxRecLine data start    
    P1X: Double;
    P1Y: Double;
    
    P2X: Double;
    P2Y: Double;
  End;  
  
{% endhighlight %}  

Records don't always occupy the memory size equivalent to the size of their component values, so adding the size allow receivers to read the correct amount of data when reading streams. There is another issue to consider: data members and records themselves have the data aligned to certain size boundaries. Since all systems reading the memory need a clear understanding of the data, we need to understand how this data can be represented in an unambiguous way. The `Size` value can be read by both parties as a first line of defense to know that records are represented in a similar way, but for complete safety we need to ensure that our data alignment matches.

## Record Alignment and Padding ##

Records are usually [padded and aligned](https://en.wikipedia.org/wiki/Data_structure_alignment), this means they have their data members and the record as a whole are padded to certain size multiples. This speeds up memory access and indexing of data. We could declare our records as `Packed` to prevent alignment or we could use explicit compiler defines (`$A` or `$ALIGN`) to set the alignment. As long as all parties involved in the transfer of the data have the same understanding of the data structure the data can be read correctly.

Records are by aligned by the smaller of :
* The record aligment setting. By default 8-byte (Quad Word)
* The size of the largest element
  
Record fields are aligned by the smaller of :
* The alignment of fields of its own size
* The aligment of the record (see above)

By placing larger members before smaller ones we could create a more compact structure because we would eliminate data member alignment requirements. This is true because data types are usually multiples of smaller types (1,2,4,8 bytes). 8 Byte alignment is the same standard used by the Windows SDK and the reason why we could do a verbatim translation of the C++ struct to a Delphi record.

If we now look at our header record the `RecType: TxRectTypeEnum` gains no space benefit defined as a type with a size of 1 byte. By default enumerated types are byte, but since our `Size` is `DWord` the record will be aligned by at least 4 bytes. This means that the size of the record will be 8 instead of 5. In our Line and Arc records it means that a pad of 3 bytes will be added before `CenterX` and before `P1X` in `TxRecArc` and `TxRecLine` respectively. Similarly since our record will be aligned to a boundary of 8 bytes (since it contains `Double`), a pad of 7 bytes is added to the end.

In memory our Record really looks like this (I added comments where padding is done)

{% highlight pascal %}
TxRecArc = Record
  Size: DWord;       // 4 bytes. Offset 0
  RecType: TxRectTypeEnum; // = TxRectType_Arc //1 byte //Offset 4
 
  //Pad1 : Array[0..2] of Byte;  // implicit pad added to align CenterX to aling to a multiple of 8 bytes
  CenterX: Double; // offset 8
  CenterY: Double;
  StartAng: Double;
  EndAng: Double;
  Radius: Double;
  
  CCW: boolean; // 1 byte. offset 48
  //Pad2 : Array[0..6] of Byte;  // implicit pad added to align record with the next record
End;
{% endhighlight %}  


Since enumerated types are typically 32-bit in other programming languages we might as well set our `TxRectTypeEnum` to be 4 bytes using (the `$Z` compiler directive). Doing so this `Pad1` will disappear and we will be more compatible with C++. Also, since we have padding after CCW we could use `WordBool` or `LongBool` datatypes without growing our record footprint

{% highlight pascal %}
{$Z4}
  TxRectTypeEnum = (TxRectType_Undefined, TxRectType_Line, TxRectType_Arc);
{$Z1}
{% endhighlight %}  

As long as we use "standard" alignment rules (8 Byte) we remain compatible with Windows SDK standards. Most applications can easily be configured to use our records and we won't have surprises about alignment of fields or records.

I highly recommend staying within the 8-Byte (Quad Word) record alignment scheme. It is well known and widely used. As long as all parties use this alignment the fields will be aligned as expected.

## Section Conclusion ##
Pointers are numeric values. Strongly typed pointers allow us to understand the structure and values at the address held by a pointer. Casting a pointer allows for a different or extended interpretation of the data at a memory address. As long as two parties have common interpretation of the structure of memory data can be easily read or written by either.

In the next section I will expand on pointer math and the transmission of arrays of data.


