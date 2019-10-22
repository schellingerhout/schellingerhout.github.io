---
title: "Active Object Design Pattern in Delphi (Part 2): The Scheduler (Activation Queue)"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi  
  - Concurrency
  - Active Object

sidebar:
  nav: activeobject  
---

In this blog post I will continue to describe the process to develop a solution in Delphi using the Active Object Design Pattern. Specifically, we will see how method requests can be scheduled

<!--more-->
If you missed [Part 1: Method Requests]({{ site.baseurl }}{% post_url 2017-11-22-activeobject-pattern1 %}) I would recommend that you read that first before proceeding.

I want to delve into more detail of the Active Object's Activation Queue or Scheduler.

The Proxy enqueues methods for the servant (passive) object and the Scheduler dequeues and invokes these methods against the Servant. Methods are enqueued in the Proxy (and Client) thread, and methods are dequeued in the Scheduler's own thread and invoked in that thread as well. As we saw in the previous post, the calls can be closures that we use to provide values to the Future Value objects held by the client.

{% highlight pascal %}
TActivationScheduler = class(TThread)
private
  FDataReady: TEvent;
  FGuardedMethodRequests: TList<TMethodRequest>;
  FActivationQueue: TQueue<TMethodRequest>;
  procedure CallPreviouslyGuarded;
protected
  // Dispatch the Method Requests on their Servant
  // in the Scheduler’s thread.

  procedure Execute; override;
public

  constructor Create;
  destructor Destroy; override;

  // Insert the Method Request into
  // the Activation_Queue. This method
  // runs in the thread of its client, i.e.,
  // in the Proxy’s thread.

  procedure enqueue(AMethodRequest: TMethodRequest);

end;
{% endhighlight %}

The Activation scheduler run's execution in its own thread while new method requests are enqueued in the client's context thread. I use a simple generic queue and protect it with a Monitor.

Some method requests may have Guards, this means that the method may only be executed when the Guard returns true. The sample provided in [Lavender and Schmidt's Paper](http://www.cs.wustl.edu/~schmidt/PDF/Act-Obj.pdf) had an implementation of a Queue as a bounded buffer. Their implementation would enumerate the queue, check guards for actions, dequeue those where guards return true and then call their methods.

With my implementation of a Monitor on the Queue the use of an enumerator does not work well. I want to keep locks as short as possible, so I rather implemented a second private list of method requests that have failed to pass the Guard.  I check this list when new actions are added (and the FDataReady event is set) and I also re-check the list when any action is successfully executed.

{% highlight pascal %}
constructor TActivationScheduler.Create;
begin
  inherited Create(False);
  FreeOnTerminate := true;

  FDataReady := TEvent.Create();
  FActivationQueue := TQueue<TMethodRequest>.Create;
  FGuardedMethodRequests := TList<TMethodRequest>.Create;
end;

destructor TActivationScheduler.Destroy;
begin
  FDataReady.SetEvent;
  FDataReady.Free;
  FActivationQueue.Free;
  FGuardedMethodRequests.Free;

  inherited;
end;

procedure TActivationScheduler.enqueue(AMethodRequest: TMethodRequest);
begin
  system.TMonitor.Enter(FActivationQueue);
  try
    FActivationQueue.enqueue(AMethodRequest);
  finally
    system.TMonitor.Exit(FActivationQueue);
  end;

  FDataReady.SetEvent;
end;

procedure TActivationScheduler.CallPreviouslyGuarded;
var
  i: integer;
  LMethodRequest: TMethodRequest;
begin
  // deal with previously guarded requests
  for i := 0 to FGuardedMethodRequests.Count - 1 do
  begin
    LMethodRequest := FGuardedMethodRequests[i];
    if LMethodRequest.guard then
    begin
      LMethodRequest.call;
      LMethodRequest.Free;
      FGuardedMethodRequests[i] := nil;
    end;
    if Terminated then
      exit;
  end;
  FGuardedMethodRequests.Pack;
end;

procedure TActivationScheduler.Execute;
var
  LMethodRequest: TMethodRequest;
  i, LCount: integer;
begin
  inherited;

  while not terminated do
  begin

    FDataReady.WaitFor();

    CallPreviouslyGuarded;

    system.TMonitor.Enter(FActivationQueue);
    try
      LCount := FActivationQueue.Count;
    finally
      system.TMonitor.Exit(FActivationQueue);
    end;

    for i := 1 to LCount do
    begin
      system.TMonitor.Enter(FActivationQueue);
      try
        LMethodRequest := FActivationQueue.Dequeue;
      finally
        system.TMonitor.Exit(FActivationQueue);
      end;

      if not LMethodRequest.guard then
      begin
        FGuardedMethodRequests.Add(LMethodRequest);
        LMethodRequest := nil;
      end;

      if LMethodRequest <> nil then
      begin
        LMethodRequest.call;
        // we need to check if the call lifted any guards
        CallPreviouslyGuarded;
      end;
      if Terminated then
        exit;
    end;

    system.TMonitor.Enter(FActivationQueue);
    try
      if FActivationQueue.Count = 0 then
        FDataReady.ResetEvent;

    finally
      system.TMonitor.Exit(FActivationQueue);
    end;
  end;
end;
{% endhighlight %}

In a later post you will see how we compose the Scheduler with the Proxy, but for now its sufficient to know that the Proxy will schedule request for its servant on the Scheduler. Events will be fired in order as Guards allow.

You can download the [source code](https://github.com/schellingerhout/active-object-delphi). Please comment or contribute.
