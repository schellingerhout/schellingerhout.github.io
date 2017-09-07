---
title: "Gang-of-Four Creational Design Pattern Examples in Delphi"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi
  - Design Patterns
---
Delphi versions of the Maze code samples of Creational Design Patterns in "Design Patterns: Elements of Reusable Object-Oriented Software". 

<!--more-->

## Abstract Factory

The abstract factory is a class that creates components and returns them to the caller as their abstract types. In this case I chose interfaces over the abstract classes as used in the samples of the book. Any number of concrete classes can implement the factory interface. The consumer in this case does not see the implementation, but just the resultant products.

{% highlight pascal %}

unit IntfMazeFactory;

interface
uses
  system.generics.collections;

type

IMaze = interface
 // definition not important for illustration
end;

IWall = interface
 // definition not important for illustration
end;

IRoom = interface
 // definition not important for illustration
end;

IDoor = interface
 // definition not important for illustration
end;

IMazeFactory = interface
  function MakeMaze: IMaze;
  function MakeWall: IWall;
  function MakeRoom(ANumber: integer): TArray<IRoom>;
  function MakeDoor(AFromRoom, AToRoom: IRoom): IDoor;  
end;

//We can pass the factory as a parameter for example
//   MazeGame.CreateMaze(AFactory);
//The game in turn can create the maze in an abstract way. 

implementation

end.
{% endhighlight %}


## Builder

The builder pattern is similar except that the consumer only sees the highest level of abstraction. Intermediate products are identified in parameters via some kind of index or other key identifier.

{% highlight pascal %}

unit IntfMazeBuilder;

interface

type

IMazeBuilder = interface
  procedure BuildMaze;
  procedure BuildRoom(ANumber: integer);
  procedure BuildDoor(AFromRoomIndex, AToRoomIndex: integer); 

  function GetMaze: IMaze; 
end;

//We can pass the builder as a parameter for example
//   LMaze := MazeGame.CreateMaze(ABuilder);
//The game in turn can create the maze in an abstract way. 
//Difference here between the builder and the factory is that the 
// Consumer does not need to know constituant parts
 
implementation

end.
{% endhighlight %}

Notice that the builder offers a higher level of abstraction 

## Factory Method

In short its a method that creates a product, the method must be overriden by decendents to extend the base (or sometimes fail-over) product range.

{% highlight pascal %}
unit IntfFactoryMethod;

interface

type

IProduct = interface
 // definition not important for illustration
end;


TAbstractProductCreator = class abstract
public
  function CreateProduct(AProductID: integer): IProduct; virtual;  
end;

//The function that creates the products is virtual and can be overridden
// The abstract creator may have an implementation or we may also have
// a base product creator defined. The design pattern really only centers 
// around the method. It must be virtual, Descendents implement and fall throug
//  is handled via inheritence

implementation

uses    
    ProductTags; //assume product IDs are defined here

Type

TBaseProductA = class(TInterfacedObnject, IProduct)
 // definition not important for illustration
end;

TBaseProductB = class(TInterfacedObnject, IProduct)
 // definition not important for illustration
end;


function TAbstractProductCreator.CreateProduct(AProductID: integer): IProduct;
begin
    // Case statement against AProductID. Descendants should add their own cases and call inherited as a fall through
   Case AProductID of 
     ProductTags.BaseProductA:
        result := TBaseProductA.Create;
     ProductTags.BaseProductB:
        result := TBaseProductB.Create;
    else
        raise EProductIDUnkownException.Create;
   end;
end;

end.
{% endhighlight %}

We can add a concrete implementation. We can add any number underneath the abstract creator. Products at the abstract creator level are always available while. Concrete factories will add any number of new products in potentially independent hierarchies. This may pose some risk in requestion products the class cannot create. It also opens up the potential of re-implementation or hiding of base products.

{% highlight pascal %}
unit ConcreateFactoryMethod;

interface
uses
    intfFactoryMethod;

type

TConcreteProductCreator = class(TAbstractProductCreator)
public
  function CreateProduct(AProductID: integer): IProduct; override;  
end;

//The function that creates the products is virtual and can be overridden at least from 

implementation

uses    
    ProductTags; //assume product IDs are defined here

Type

TAdvancedProductX = class(TInterfacedObnject, IProduct)
 // definition not important for illustration
end;

TAdvancedProductY = class(TProductA)
 // definition not important for illustration
end;


function TConcreteProductCreator.CreateProduct(AProductID: integer): IProduct;
begin
   Case AProductID of
     
     ProductTags.AdvancedProductX:
        result := TAdvancedProductX.Create;
     ProdcutTags.BaseProductA,  ProductTags.AdvancedProductY: //We may hide product creation or introduce new ones
        result := TAdvancedProductY.Create;
    else
      result := inherited;
   end;
end;

end.
{% endhighlight %}

## Singleton

I rarely implement the singleton pattern and rather I prefer a pure abstract class with class properties and class methods. This pattern is particularly tricky in Delphi because you always have at least the constructor at the base TObject level, so it cannot be implemented as defined in the book. The only way to fully hide the constructor is to supply an interface via a class.

{% highlight pascal %}
unit SingletonMazeFactory;

interface
uses
  IntfMazeFactory;


Function MazeFactoryInstance: IMazeFactory;

implementation

type

TMazeFactory = class(TInterfacedObject, IMazeFactory)
private 

private
  class var FInstance: IMazeFactory; 

  class function GetInstance: IMazeFactory; static;
public
  Constructor Create;

  function MakeMaze: IMaze;
  function MakeWall: IWall;
  function MakeRoom(ANumber: integer): TArray<IRoom>;
  function MakeDoor(AFromRoom, AToRoom: IRoom): IDoor;  

  class property Instance: IMazeFactory read GetInstance;
end;


Function MazeFactoryInstance: IMazeFactory
begin
  result := TMazeFactory.Instance;
end;

class function TMazeFactory.GetInstance: IMazeFactory;
begin
  if FInstance = nil then
    FInstance := TMazeFactory.Create;
  result := FInstance; 
end;

function TMazeFactory.MakeMaze: IMaze;
begin
   // definition not important for illustration
end;

constructor TMazeFactory.Create;
begin
  inherited Create;
end;

function TMazeFactory.MakeWall: IWall;
begin
   // definition not important for illustration
end;

function TMazeFactory.MakeRoom(ANumber: integer): TArray<IRoom>;
begin
   // definition not important for illustration
end;

function TMazeFactory.MakeDoor(AFromRoom: IRoom; AToRoom: IRoom): IDoor;
begin
   // definition not important for illustration
end;

end.
{% endhighlight %}

## Prototype
This is one of my favorite design patterns. To use it you provide prototype instances during construction. These will then be used to create clones in the factory. This provides extensibility where you have no access to the code (like a user-defined extension or plug-in).

{% highlight pascal %}
unit IntfMazeFactory;

interface
uses
  system.generics.collections;

type

IMaze = interface
  function Clone: IMaze;
 // rest of definition not important for illustration
end;

IWall = interface
  function Clone: IWall;
 // rest of definition not important for illustration
end;

IRoom = interface
  function Clone: IRoom;

 // rest of definition not important for illustration
end;

IDoor = interface
  function Clone: IDoor;
  procedure Initialize(AFromRoom, AToRoom: IRoom); //mutator
 // rest of definition not important for illustration
end;

TMazePrototypeFactory = Class 
private
  FProtoMaze: IMaze; 
  FProtoWall: IWall; 
  FProtoRoom: IRoom; 
  FProtoDoor: IDoor;
public
  constructor Create(AMaze: IMaze; AWall: IWall; ARoom: IRoom; ADoor: IDoor);

  function MakeMaze: IMaze;
  function MakeWall: IWall;
  function MakeRoom(ANumber: integer): TArray<IRoom>;
  function MakeDoor(AFromRoom, AToRoom: IRoom): IDoor;  
end;

//We can pass the factory as a parameter for example
//   MazeGame.CreateMaze(AFactory);
//The game in turn can create the maze in an abstract way. 

implementation

constructor TMazePrototypeFactory.Create(AMaze: IMaze; AWall: IWall; ARoom: IRoom; ADoor: IDoor);
begin
  inherited Create;
  FProtoMaze := AMaze; 
  FProtoWall := AWall; 
  FProtoRoom := ARoom; 
  FProtoDoor := ADoor;
end;

function TMazePrototypeFactory.MakeMaze: IMaze;
begin
  result := FProtoMaze.Clone;
end;

function TMazePrototypeFactory.MakeWall: IWall;
begin
  result := FProtoWall.Clone;
end;

function TMazePrototypeFactory.MakeRoom(ANumber: integer): TArray<IRoom>;
begin
  for i := 1 to ANumber do
    result.Add(FProtoRoom.Clone);
end;

function TMazePrototypeFactory.MakeDoor(AFromRoom: IRoom; AToRoom: IRoom): IDoor;
begin
  result := ARoom.Clone;
  result.Initialize(AFromRoom, AToRoom);
end;

end.
{% endhighlight %}

