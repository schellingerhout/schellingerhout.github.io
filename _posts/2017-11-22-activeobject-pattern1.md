---
title: "Active Object Design Pattern in Delphi (Part 1): Method Requests and Future Values"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi  
  - Concurrency
  - Active Object
---

In this blog post I will start to describe the process to develop a solution in Delphi using the Active Object Design Pattern. This pattern is a concurrency design pattern based in part on the Proxy pattern. 

<!--more-->
As far as I can determine it originated from "Pattern-Oriented Software Architecture, Volume 2: Patterns for Concurrent and Networked Objects" and you can view an updated paper on the subject here [](http://www.cs.wustl.edu/~schmidt/PDF/Act-Obj.pdf). I will refer to the paper from time to time and it may be good to first read it to grasp what it entails.

The Active Object Pattern describes a system where the Client interacts with a Proxy that directs commands to a Servant Object. The Proxy presents the same or similar interface as the Servant to consumers. The Proxy uses a Scheduler (or Activation Queue) to queue method requests to the Servant and dispatches them in order*. Methods with return values are returned as Futures (also known as Promises). Demanding a Future Value blocks until the value is ready.

Every call to the Active Object gets turned into a Method Request. So let us start there

{% highlight pascal %}
  TMethodRequest = class
    // some details covered later
  public
    function Guard: boolean; 
    procedure Call; 
  end;
{% endhighlight %}

The method request just like we saw in Lavender and Schmidt's paper has a guard that determines if the call can be be made and a call to invoke the actual method on the client. The sceduler loops through the queued requests, checks their Guard and then issues a Call.

Some mehtod requests need to satisfy Futures or Promised Values. Instead of the standard property "getters" in the Servant, the Proxy should generally return the value in a Future, so that the client can delay the use of the value for as long as possible since it may be a blocking action if the value is not ready. To facilitate with the return of Future Values consider this simple generic interface  


{% highlight pascal %}
  IFutureValue<T> = interface
    function GetValue: T;
    property Value: T read GetValue;
  end;
{% endhighlight %}

:Note: Delphi has a IFuture<T> that is run via a TTask, that interface is more complex than we need to implement.  

 In the sample presented in the paper each method request decends from the abstract method request and are constructed using the Servant and - in cases returning a value - a Future or Promise that needs to be satisfied.  I recreated a skeletal structure representing the design of the sample in the paper below. The main difference here is that the Future is created with a Method Request. In the paper the Proxy creaded the Future and passed it into the Method Request to be filled.

{% highlight pascal %}
 TMethodRequest = class abstract
 public
    function guard: boolean; virtual; abstract;
    procedure call; virtual; abstract;
 end;

// The sample only had one future. 
IMessage_Future = Interface
  function GetValue: TMessage;
  property Value : TMessage read GetValue;
end;

// each method invocation is a class
// The sample only had a Put and a Get methods to implement

 TPut = class(TMethodRequest)
 private
   FServant : TPassiveObject;
   FArg : TMessage;
 public
   constructor Create(ARep: TPassiveObject; AArg: TMessage);

   function guard: boolean; override;  // result := not FServant.full_i;
   procedure call; override;  // FServant.put_i(FArg);
 end;


TGet = class(TMethod_Request)
public type
   TMessage_Future = class(TInterfacedObject,  IMessage_Future)
      // detail removed
   end;

 private
   FServant : TPassiveObject;
   FFuture: IMessage_Future;
 public
   constructor Create(ARep: TPassiveObject); // creates the Future Value. 
                                             // Client can read it from the Future property  
                                             // and hold it for delayed evaluation

   function guard: boolean; override;
   procedure call; override; // TMessage_Future(FFuture).SetValue(FServant.get_i);  

   property Future:   IMessage_Future read FFuture;
end;
{% endhighlight %}

And the Proxy would use the objects as follows

{% highlight pascal %}
procedure TProxy.put(const msg: TMessage);
var
  LPut: TPut;
begin
  LPut := TPut.Create(FServant, msg);
  FScheduler.enqueue(LPut);
end;

function TProxy.get: IMessage_Future;
var
  LGet: TGet;
begin
   LGet := TGet.Create(FServant);
   result := LGet.Future;
   FScheduler.enqueue(LGet);
end;
{% endhighlight %}

** Generalizing the sample

While this somewhat direct conversion of the sample from the paper would work in Delphi there are a few problems with the design. First the sample is very simple. Only two method invocations with one type passed. There is only on Future Value type and lastly the Method requests are tightly bound to the Passive objects on which they operate.

We can use anonymous functions and closures to generalize the Method Request class

{% highlight pascal %}
  TMethodRequest = class
  private
    FCall: TProc;
    FGuard: TFunc<boolean>;
    FGuardless: boolean; // set to Assigned(FGuard) in constructor  
  public
    constructor Create(const ACall: TProc; const AGuard: TFunc<boolean> = nil);

    function guard: boolean;  // returns true if Guardless else invokes FGuard
    procedure call; virtual;  // invokes FCall
  end;
{% endhighlight %}

We can pass a simple TProc that is invoked in TMethodRequest.call via the Scheduler. We can also pass a Guard that can determine if the call can be made or needs to wait. 


{% highlight pascal %}
procedure TPassiveObjectProxy.put(const msg: TMessage);
var
  LMsg: TMessage;
begin
  LMsg := msg;
  FScheduler.Equeue(
    TMethodRequest.Create(
      // Call
      procedure
      begin
        FServant.put_i(LMsg);
      end,

      // Optional Guard
      function : boolean
      begin
        result := not FServant.empty_i;
      end
    )
  );
end;
{% endhighlight %}

We no longer need to subclass TMethodRequest, but how we we return Futures? First, let us define the implementation of of our TFutureValue<T> generic class:

{% highlight pascal %}
  TFutureValue<T> = class(TInterfacedObject, IFutureValue<T>)
  private
    FResult: T;
    function GetValue: T;
    // synchronization details omited
  public
    procedure SetValue(AResult: T); // called when method request is invoked
    property Value: T read GetValue;
  end;
{% endhighlight %}

Our code on the Proxy side is a bit more code, but it is still better than creating a new MethodRequest sub-class

{% highlight pascal %}
function TPassiveObjectProxy.get: IFuture<TMessage>;
var
  LActiveFuture: TFuture<TMessage>;
begin
  LActiveFuture := TFutureValue<TMessage>.Create;
  result := LActiveFuture;

  FScheduler.Enqueue(
    TMethodRequest.Create(
      // Call
      procedure
      begin
        LActiveFuture.SetValue(FServant.get_i); // closure over the future and servant
      end,

      // Optional Guard
      function : boolean
      begin
        result := not FServant.empty_i;
      end
    )
  );
end;
{% endhighlight %}

The above looks like more work, but consider the case where the Servant had many more methods or many more properties. We would have to create classes for each of those. Say my Servant object has another property PropA of type integer we can easily make could quickly add a setter and getter to the Proxy

{% highlight pascal %}
function TPassiveObjectProxy.getPropAFuture: IFuture<integer>;
var
  LActiveFuture: TFuture<integer>;
begin
  LActiveFuture := TFutureValue<integer>.Create;
  result := LActiveFuture;

  FScheduler.Enqueue(
    TMethodRequest.Create(
      // Call without a Guard
      procedure
      begin
        LActiveFuture.SetValue(FServant.PropA); // closure over the future and servant
      end
    )
  );
end;

procedure TPassiveObjectProxy.SetPropA(const Value: Integer);
var
  LValue: Integer;
begin
  LValue := Value;
  FScheduler.Equeue(
    TMethodRequest.Create(
      // Call without a Guard
      procedure
      begin
        FServant.propA := LValue;
      end
    )
  );
end;

// For backwards compatibility only... better for clients to use the Future
procedure TPassiveObjectProxy.GetPropA: integer;
begin
  result:=  GetPropAFuture.Value;
end;
{% endhighlight %}


Why did I not simply use Tasks and Futures from the System.Threading library?  You can create a Task without running it immediately and with a bit of effort you can create a Future that is not started immediately so perhaps I could have run those as the scheduler dequeues them. I initially went down that path, but the code was actually more complex and more unreadible. I needed a simple solution that checks a guard and calls a method. While most methods do not need guards, it is part of the pattern and was hard to implement with the out-of-the-box structures

You can download the [source code](https://github.com/schellingerhout/active-object-delphi). Please comment or contribute.


