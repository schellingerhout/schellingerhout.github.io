---
title: "Active Object Design Pattern in Delphi (Part 4): The Servant and the Proxy"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Delphi  
  - Concurrency
  - Active Object
---

In this blog post I will continue to describe the process to develop a solution in Delphi using the Active Object Design Pattern. Specifically, we will see how the Proxy composes and uses a Scheduler and a Servant

<!--more-->
If you missed [Part 1: Method Requests]({{ site.baseurl }}{% post_url 2017-11-22-activeobject-pattern1 %}), [Part 2: The Scheduler]({{ site.baseurl }}{% post_url 2017-11-24-activeobject-pattern2 %}) or [Part 3: Futures]({{ site.baseurl }}{% post_url 2017-11-25-activeobject-pattern3 %}). I would recommend that you read those first before proceeding. 

In the fourth part of the deep dive into the Active Object pattern I want to look at how we put the pieces together to create a Proxy for a Servant object. First a few guidelines given by the pattern:

  - The Servant does not do synchronization, it is unaware of the Scheduler and the Proxy
  - The Client should be able to use the Proxy in the same or similar way to the Servant (except for Futures) 
  - The Client and Proxy operate in the Client's thread, the Scheduler and Servant operate in the Scheduler's thread

The Servant is an ordinary object that holds the implementation or functions of the work and data that need to grouped together. If we are using the Active Object Pattern to retrofit an existing system, the Servant would be your current non-threaded object that you would want to be running in its own thread.

The Proxy presents the same or a very similar interface to the Client as the Servant would have. This allows us to retrofit our code to use an object that runs in its own thread without any special synchronization code in the Client. The Proxy typically internally constructs the Servant as well as the Scheduler and would enqueue methods intended for the servant 

Here is an example of a Servant that provides methods `put_i`, `get_i`, `empty_i` and `full_i`. The naming was chosen to match the sample provided in [Lavender and Schmidt's Paper](http://www.cs.wustl.edu/~schmidt/PDF/Act-Obj.pdf). The message type (here called `TMessage`) is immaterial to the demonstration.  Assume that it is communication that gets placed or read from the queue.

{% highlight pascal %}
TServant = class
private
   // Internal Queue representation, e.g., a
  // circular array or a linked list, etc.

public
  constructor Create(mqsize: cardinal);

  // Message queue implementation operations.
  procedure put_i(const msg: TMessage);
  function get_i: TMessage;

  function empty_i: boolean;
  function full_i: boolean;

end;
{% endhighlight l %}

I won't cover `TServant`'s implementation because it simply enqueues and dequeues in a simple queue. It has no synchronization knowledge and does not need to have any special code to be thread safe or deal with concurrency.

The proxy will present a similar interface as the Servant. I assume that the sample in the paper used  `*_i` for methods of the Servant to distinguish those in the Proxy that are named without the `_i` (perhaps meaning "internal"). However, in a real example the Proxy may actually follow the "Decorator" pattern or implement the same interface(s) as the Servant.

The Proxy composes the Servant and the Scheduler as follows

{% highlight pascal %}
TProxy = class
private
 
protected
    // The Servant that implements the
    // Active Object methods.
    FServant: TServant;

   // A scheduler for the Message Queue.
    FScheduler: TActivationScheduler;

public
  constructor Create; // Also creates composed FServant and FScheduler
  destructor Destroy; override;

  procedure put(const msg: TMessage);
  function get : IFuture<TMessage>;

  // empty and full may also need implementation

end;
{% endhighlight %}

And the implementation would look like the sample we saw in [Part 1: Method Requests]({{ site.baseurl }}{% post_url 2017-11-22-activeobject-pattern1 %}). Our `put` and `get` are converted to Method Requests and enqueued with the Scheduler.

{% highlight pascal %}
procedure TProxy.put(const msg: TMessage);
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
        result := not FServant.full_i;
      end
    )
  );
end;

function TProxy.get: IFuture<TMessage>;
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


This concludes the series on the Active Object pattern. I hope you found it useful. You can download the [source code](https://github.com/schellingerhout/active-object-delphi). Please comment or contribute.
