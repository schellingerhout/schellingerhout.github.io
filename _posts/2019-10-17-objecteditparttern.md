---
title: "A Successful Object Edit Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi
 
sidebar:
  nav: objecteditpattern  
  
---
A successful Object-Oriented Design Pattern for editing objects. This pattern allows free editing of objects, while allowing cancellation and preservation of the original object reference.
<!--more-->

I will guide you through my thought processes in developing a code pattern that I have used successfully in multiple implementations.

The goals of the object edit pattern is as follows: 
* Editing an object should allow free manipulation of any of its values
* Canceling an edit should leave the original object unchanged
* Committing an edit should leave the original object's pointer or reference unchanged

I will present this pattern using Delphi, but the concepts transcend any single programming language.

## Risk-free Editing of Object ##

This goal proves to be a challenge in that we may manipulate any number of properties at any number of levels. The implementation of an undo system may be complex. The use of a light weight editable object is brittle and requires constant maintenance. The simplest solution to this problem is to edit an identical independent copy of the original object. 

To facilitate editing copies, each of my editable objects will need to have a copy constructor (here named `CreateCopy`):

{% highlight pascal %}
type

TABC = Class(TBase)
public
  Constructor Create; 
  Constructor CreateCopy(AABC: TABC); 
  Destructor Destroy; override;
  
  function Clone: TBase; override;  
end;

{% endhighlight %}

We can also add a virtual method `Clone` at the base class that each class can override. Adding a virtual method that accesses the copy constructor is handy in cases where we don't need to deal with specific types, such as cases where we need to duplicate all the elements of a list. Its implementation would be:

{% highlight pascal %}
function TABC.Clone: TBase;
begin
  result := TABC.CreateCopy(self);
end;
{% endhighlight %}


## Canceling leaves original object unchanged ##

This goal is already accomplished with our editing of a copied object, provided that we implement our copy constructor according to the following rules:

* Composed objects are duplicated with a copy constructor. 
* Weak references to values of reference types are assigned directly. This includes
  * object and pointer types
  * reference counted types: strings, dynamic arrays, interfaces

This new requirement means that all composed objects also need a copy constructor. Composed lists can have their elements duplicated with the use of the `Clone` virtual method, instead of checking each class and calling each specific copy constructor by type. The only problem situation is composed interfaced types that are edited along with the object. This odd scenario would require special handling, I consider this an exceptional case and is not addressed by this pattern. 

## Committing Edits Leaves the Original Object Pointer Unchanged ##
  
The safety provided by editing the copied object also introduces a problem. We have a new object, with a new reference. All references to the original object will need to be updated. This can be resolved by "assigning" the edited copy to the original object. An assignment method that takes an object and assign its fields, references and composed objects according to similar rules to our copy constructor will serve this purpose. We can even use a properly defined assignment method in our copy constructor to prevent duplication of code. Our copy constructor would then simply look like this.

{% highlight pascal %}
procedure TABC.CreateCopy(AABC: TABC); 
begin
  Create; // NOT inherited create
  self.Assign(AABC);
end;
{% endhighlight %}

Our base class implementation could look like this

## Assignable class implementation ##
{% highlight pascal %}
TBase = Class
  procedure Assign(ASource: TBase); virtual; abstract;
  function Clone: TBase; virtual; abstract;
end;
{% endhighlight %}

And descendants like this

{% highlight pascal %}
TDescendant = Class(TBase)
begin
  Constructor Create; 
  Constructor CreateCopy(ASource: TDescendant);
  Destructor Destroy; override;

  procedure Assign(ASource: TBase); overide;
  function Clone: TBase; virtual; abstract;  
end;
{% endhighlight %}

Thus, our complexity is shifted from our copy constructor and clone methods towards our implementation of an assignment method.

Here is an example of what our `Assign` method could look like on a complex object

{% highlight pascal %}
procedure TMyComplexObject.Assign(ASource: TBase);
var
 LSource: TMyComplexObject;
begin
  inherited;
  if ASource is TMyComplexObject then
  begin
    LSource := TMyComplexObject(ASource);
    FField1 := LSource.FField1;
    FComposedObject.Assign(LSource.FComposedObject);
    FComposedList.Assign(LSource.FComposedList);
    TListHelper.Assign(FNonAssignableList, LSource.FNonAssignableList); 
    FWeakReference := LSource.FWeakReference;  
  end; 
end;
{% endhighlight %}

Besides the rules of copy constructors, here re-appropriated for assignment there are a few best practices:
* Assign fields instead of properties. We should be concerned with data preservation, properties may have side-effects
* Composed objects must allow assignment
* Composed lists must allow assignment or you will need to create classes that can facilitate deep copying


Delphi provides a standard base class called `TPersistent` that provides a public virtual method `Assign` and a protected virtual counterpart `AssignTo`.  We could use that as a base if desired, but there is no clear of convincing argument to use it, except in the case of requiring decendency from this class (for example serialization purposes)

{% highlight pascal %}
TBase = Class(TPersistent)
  procedure Assign(ASource: TPersistent);  overide;
  function Clone: TBase; virtual; abstract;  
end;

...

procedure TBase.Assign(ASource: TPersistent);
begin
  if not (ASource is TBase) then
    inherited; // will raise an exception 
end;

{% endhighlight %}

Decendent classes would look like this:

{% highlight pascal %}
TDescendant = Class(TBase)
begin
  Constructor Create; 
  Constructor CreateCopy(ASource: TDescendant);
  Destructor Destroy; override;

  procedure Assign(ASource: TPersistent);  overide;
  function Clone: TBase; virtual; abstract;  
end;
{% endhighlight %}

## Our Editor ##

Once we have our class with copy constructor and assignment method our implementation of an editor is quite trivial.

{% highlight pascal %}
Interface
Type

TABCEditor = class
  class function Edit(const ASubject: TABC) : TModalResult; static;
end;


implementation
...

class function TABCEditor.Edit(const ASubject: TABC) : TModalResult;
var
  LABCEditDialog: TABCEditDialog;
  LEdited: TABC;
begin 
  LABCEditDialog := TABCEditDialog.Create(nil);
  LEdited := TABC.CreateCopy(ASubject);
  try
    LABCEditDialog.ABC := LEdited;
    result := LABCEditDialog.ShowModal;
  
    if result = mrOK then
      ASubject.Assign(LEdited);
  
  finally
    LEdited.Free;
    LABCEditDialog.Free;
  end;
end;

{% endhighlight %}

Our edit dialog can manipulate the object at will, knowing that if we hit cancel no edits are preserved, and if we commit our changes, we will assign them to the original object.

## Conclusion ##

I hope you find the code pattern useful. If you do, please be so kind to link to this article
