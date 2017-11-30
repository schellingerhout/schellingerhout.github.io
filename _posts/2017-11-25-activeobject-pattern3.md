---
title: "Active Object Design Pattern in Delphi (Part 3): Futures (Promises)"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi  
  - Concurrency
  - Active Object
---

In this blog post I will continue to describe the process to develop a solution in Delphi using the Active Object Design Pattern. Specifically, we will see how Future Values are delivered

<!--more-->
If you missed [Part 1: Method Requests]({{ site.baseurl }}{% post_url 2017-11-22-activeobject-pattern1 %}) or [Part 2: The Scheduler]({{ site.baseurl }}{% post_url 2017-11-24-activeobject-pattern2 %}). I would recommend that you read those first before proceeding.

In the third part of the deep dive into the Active Object pattern I want to look at the future value. Unlike the IFuture that is a Task in Delphi, this form of future is simple a container that waits to be filled and triggers the event to release once filled. 

We have already looked at the interface we would need in [Part 1: Method Requests]({{ site.baseurl }}{% post_url 2017-11-22-activeobject-pattern1 %}). For reference, here it is again

{% highlight pascal %}
IFutureValue<T> = interface
  function GetValue: T;
  property Value: T read GetValue;
end;
{% endhighlight %}

Below is the class defininition for the Future Value. 

{% highlight pascal %}
TFutureValue<T> = class(TInterfacedObject, IFutureValue<T>)
private
  FResultSet: boolean;
  FResult: T;
  FValueReadyEvent: TLightWeightEvent;   // consider balance between TEvent and TLightweight event
  function GetValue: T;
  function Wait: boolean;
  function GetValueReadyEvent: TLightWeightEvent;
protected
  property ValueReadyEvent: TLightWeightEvent read GetValueReadyEvent;
public
  destructor Destroy; override;
  procedure SetValue(AResult: T); // done by methodrequest wrapping a future
  property Value: T read GetValue;
end;
{% endhighlight %}

Lets examine some of the basics of the definition:

The property `Value` uses the `GetValue` method to `Wait` for the `FValueReadyEvent` and then returns `FResult`. The boolean `FResultSet` is used in a technique called Quick-Check Locking. I will disuss that when we look at the implementation of the class.

`SetValue` is called by the service that supplies the value (in this case MethodRequest via theScheduler). When `SetValue` is called, the `FValueReadyEvent` is set and `FResultSet` is set to `true`.

The `GetValueReadyEvent` is used to lazy load the `FValueReadyEvent` when first needed.

Here follows the implementation:

{% highlight pascal %}
destructor TFutureValue<T>.Destroy;
begin
  FValueReadyEvent.Free;
  inherited;
end;

function TFutureValue<T>.GetValue: T;
begin
  Wait;
  result := FResult;
end;

function TFutureValue<T>.GetValueReadyEvent: TLightWeightEvent;
var
  LEvent: TLightWeightEvent;
begin
  if FValueReadyEvent = nil then
  begin
    LEvent := TLightWeightEvent.Create;
    if TInterlocked.CompareExchange<TLightWeightEvent>(FValueReadyEvent, LEvent,
      nil) <> nil then
      LEvent.Free;

    if FResultSet then //can't see this happening, but here just in case
      FValueReadyEvent.SetEvent;
  end;
  result := FValueReadyEvent;
end;

procedure TFutureValue<T>.SetValue(AResult: T);
begin
  //raise exception if set twice!

  FResult := AResult; // no-one can read until we set the flag anyway, so no need for interlocked 
                      // interchange, which is hard to do with generics anyway
  FResultSet := true; //don't think interlock exchange is needed on a boolean
  GetValueReadyEvent.SetEvent; 
end;

function TFutureValue<T>.Wait: boolean;
begin
  if FResultSet then   //set after value is set
    result := true
  else
    result := ValueReadyEvent.WaitFor(INFINITE) <> TWaitResult.wrTimeout;
end;
{% endhighlight %}

For simplified synchronization the assumption must be made that the value will only be set once. This allows us to use a simple boolean that we will set directly without an interlocked exchange. This boolean is used in a technique called quick-checked locking, which is a way of reduce the overhead of checking or waiting for a synchronization object. This technique is similar to double-checked locking used in the Singleton Pattern, but is contra-indicated in some languages since order of execution is not always guaranteed. In this case I believe it is safe, and it speeds up the `GetValue` routine when the value has already been delivered

You will notice that the value is set without an interlocked exhange. I believe this is safe with the assumption that we can write only once and can read only after the TEvent is set. I also believe that setting booleans are safe and do not require an interlock exchange. If you believe I am in error please comment below or create a pull request on the source code for this blog post series.

You can download the [source code](https://github.com/schellingerhout/active-object-delphi). Please comment or contribute.
