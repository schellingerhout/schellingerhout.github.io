---
title: "Jusst a test page"
excerpt: "Jasper is just testing how Github renders the markdown"
sitemap: false
permalink: /justtesting.html
---

Nothing to see here

``` pascal
unit RecordList;

interface

uses
  System.Generics.Defaults;

Type


  TRecordList<T> = Record
  public
    Type
      RecordPointer = ^T;


    Type
      TEnumerator = class
      private
        FIndex: integer;
        FCount: integer;
        FList: TArray<T>;   //dynamic arrays are ref counted
      public
        constructor Create(count: integer; const List: TArray<T>);
        function GetCurrent: RecordPointer; inline;
        function MoveNext: Boolean; inline;
        property Current: RecordPointer read GetCurrent;
      end;


  private
    FCount: integer;
    FList: TArray<T>;
    FEqualityComparer: IEqualityComparer<T>;

    procedure Expand;
    function Get(Index: Integer): RecordPointer;
    procedure SetCount(const Value: integer);
    function GetEqualityComparer: IEqualityComparer<T>;
    function GetCapacity: integer;
    procedure SetCapacity(const Value: integer);



  public
    constructor Create(InitialCapacity: integer);

    function Add: RecordPointer; overload;
    procedure Add(const Element: T); overload;  // copy record
    procedure Add(const Elements: TArray<T>); overload;   // copy records

    function Insert(Index: integer): RecordPointer; overload;
    procedure Insert(Index: integer; const Element: T); overload;  // copy record

    procedure SetStartIndex(Index: integer);
    procedure Rotate(StartIndex, EndIndex: integer);

    procedure AddDistinct(const Element: T);  // copy record if not present

    procedure TakeOwnership(var Elements: TArray<T>);

    procedure MoveDataTo(var RecordList: TRecordList<T>);
    procedure MoveDataFrom(var RecordList: TRecordList<T>);

    function GetCopy:  TRecordList<T>;
    function GetCopyWithBuffer(MaxBuffer: integer): TRecordList<T>;

    function GetArray: TArray<T>;
    function GetCopyOfArray: TArray<T>;

    function First: RecordPointer;
    function Last: RecordPointer;

    procedure Sort(const Comparer: IComparer<T>);

    procedure Pack;
    procedure PackWithBuffer(MaxBuffer: integer);
    procedure Clear;

    procedure Exchange(Index1, Index2: integer);
    function GetEnumerator:  TEnumerator;
    procedure Remove(Index: integer);

    property Items[Index: Integer]: RecordPointer read Get; default;  // readonly!
    property Count: integer read FCount write SetCount;
    property EqualityComparer: IEqualityComparer<T> read GetEqualityComparer write FEqualityComparer;
    property Capacity: integer read GetCapacity write SetCapacity;
  End;



implementation

uses
  System.Generics.Collections;

{ TRecordList<T> }
procedure TRecordList<T>.Exchange(Index1, Index2: integer);
var
  Temp: T;
begin
  Temp := FList[Index1];
  FList[Index1] := FList[Index2];
  FList[Index2] := Temp;
end;

procedure  TRecordList<T>.Expand;
var
  Size: integer;
begin
  Size := Length(FList);
  repeat
    if Size > 64 then
      Size := (Size * 3) div 2
    else if Size > 8 then
      Size := Size + 16
    else
      Size := Size + 4;
  until Size > FCount;

  SetLength(FList, Size);
end;


function TRecordList<T>.First: RecordPointer;
begin
  result := @FList[0];
end;

function TRecordList<T>.Get(Index: Integer): RecordPointer;
begin
  result := @FList[Index];
end;

function TRecordList<T>.GetArray: TArray<T>;
begin
  if FCount <> Length(FList) then
    Pack;

  result := FList;
end;

function TRecordList<T>.GetCapacity: integer;
begin
  result := Length(FList);
end;

function TRecordList<T>.GetCopy: TRecordList<T>;
begin
  result.FList := Copy(FList, 0, FCount);
  result.FCount := FCount;
end;

function TRecordList<T>.GetCopyWithBuffer(MaxBuffer: Integer): TRecordList<T>;
var
  AvailableBuffer: integer;
begin
  AvailableBuffer := Length(FList) - FCount;
  if AvailableBuffer < MaxBuffer then
    MaxBuffer := AvailableBuffer;

  result.FList := Copy(FList, 0, FCount + MaxBuffer);
  result.FCount := FCount;
end;


function TRecordList<T>.GetCopyOfArray: TArray<T>;
begin
  result := Copy(FList);
end;

function TRecordList<T>.GetEnumerator: TEnumerator;
begin
  result := TEnumerator.Create(FCount, FList);
end;

function TRecordList<T>.GetEqualityComparer: IEqualityComparer<T>;
begin
  if FEqualityComparer = nil then
     FEqualityComparer := TEqualityComparer<T>.Default;

  result :=  FEqualityComparer;
end;

function TRecordList<T>.Insert(Index: integer): RecordPointer;
begin
  if index >= FCount then
    Exit(Add());

  if FCount = Length(FList) then
    Expand;

  if Index < FCount then
    System.Move(FList[Index], FList[Index + 1], (FCount - Index) * SizeOf(T));

  Inc(FCount);

  result :=  @FList[Index];
end;

procedure TRecordList<T>.Insert(Index: integer; const Element: T);
begin
  Insert(Index)^ := Element;
end;

function TRecordList<T>.Last: RecordPointer;
begin
  result := @FList[FCount-1];
end;

procedure TRecordList<T>.MoveDataFrom(var RecordList: TRecordList<T>);
begin
  RecordList.MoveDataTo(self);
end;

procedure TRecordList<T>.MoveDataTo(var RecordList: TRecordList<T>);
begin
  Pack;
  if RecordList.count = 0 then
    RecordList.TakeOwnership(FList)
  else
    RecordList.Add(FList);

  Clear;
end;

procedure TRecordList<T>.Pack;
begin
  SetLength(FList, FCount);
end;

procedure TRecordList<T>.PackWithBuffer(MaxBuffer: integer);
var
  AvailableBuffer: integer;
begin
  AvailableBuffer := Length(FList) - FCount;

  if AvailableBuffer <= MaxBuffer then
    exit;

  SetLength(FList, FCount + MaxBuffer);
end;

procedure TRecordList<T>.Remove(Index: integer);
begin
  Dec(FCount);

  if Index < FCount then
    System.Move(FList[Index + 1], FList[Index], (FCount - Index) * SizeOf(T));   // source, dest, count
end;

procedure TRecordList<T>.Rotate(StartIndex, EndIndex: integer);
var
  ItemAtStart: T;
  LNumMovedToStart: integer;

begin

  if EndIndex <= StartIndex then
    exit;

  ItemAtStart := FList[StartIndex]; //copy to preserve
  System.Move(FList[StartIndex + 1], FList[StartIndex], EndIndex - StartIndex);
  FList[EndIndex] := ItemAtStart;

end;

procedure TRecordList<T>.SetCapacity(const Value: integer);
begin
  if Value > Length(FList) then
    SetLength(FList, Value);
end;

procedure TRecordList<T>.SetCount(const Value: integer);
begin
  FCount := Value;
end;

procedure TRecordList<T>.SetStartIndex(Index: integer);
var
  NumMovedToStart: integer;
  FBuffer: TArray<T>;
begin

 if Index > 0 then
 begin
   SetLength(FBuffer, Index);
   System.Move(FList[0], FBuffer[0], Index * SizeOf(T));

   NumMovedToStart := FCount-Index;
   System.Move(FList[Index], FList[0], NumMovedToStart * SizeOf(T));
   System.Move(FBuffer[0], FList[NumMovedToStart], Index * SizeOf(T));
 end;

end;

procedure TRecordList<T>.Sort(const Comparer: IComparer<T>);
begin
  TArray.Sort<T>(FList, Comparer, 0, FCount);
end;

procedure TRecordList<T>.Add(const Element: T);
begin
  Add()^ := Element;
end;

procedure TRecordList<T>.Add(const Elements: TArray<T>);
var
  NextSlotIndex: integer;
begin

  NextSlotIndex :=  FCount;  // next
  Inc(FCount, Length(Elements));

  if FCount > High(FList) then
    Expand;  // Expand is based on FCount

  if Length(Elements) > 0 then
     Move(Elements[0], FList[NextSlotIndex],  Length(Elements) * SizeOf(T));

end;


procedure TRecordList<T>.AddDistinct(const Element: T);
var
  i: integer;
begin

  for i := FCount-1 downto 0 do
    if EqualityComparer.Equals(FList[i], Element) then
      exit;

  Add(Element);
end;

function TRecordList<T>.Add: RecordPointer;
begin
  if FCount > High(FList) then
    Expand;

  result := @FList[FCount];
  Inc(FCount);
end;

procedure TRecordList<T>.Clear;
const
  DefaultRec: TRecordList<T> = ();
begin
  self := DefaultRec;
end;


constructor TRecordList<T>.Create(InitialCapacity: integer);
begin
  FCount := 0;
  // FList is auto intialized since it is a dynamic array
  if InitialCapacity > 0 then
    SetLength(FList, InitialCapacity);
end;


procedure TRecordList<T>.TakeOwnership(var Elements: TArray<T>);
begin
  FList := Elements;
  Elements := nil;
  FCount := Length(FList);
end;

{ TRecordList<T>.TEnumerator<T> }

constructor TRecordList<T>.TEnumerator.Create(count: integer; const List: TArray<T>);
begin
  inherited Create;
  FIndex := -1;
  FCount := count;
  FList := List;
end;

function TRecordList<T>.TEnumerator.GetCurrent: RecordPointer;
begin
  Result := @FList[FIndex];
end;

function TRecordList<T>.TEnumerator.MoveNext: Boolean;
begin
  Inc(FIndex);
  Result := FIndex < FCount;
end;

end.
```
