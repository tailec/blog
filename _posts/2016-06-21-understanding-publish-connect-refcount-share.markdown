---
layout: post
title:  "Understanding Publish, Connect, RefCount and Share in RxSwift"
date:   2016-06-27 18:01:34
categories: swift
comments: true
---
In this article, I aim to explain Publish, Connect, RefCount and Share operators
using RxSwift.

These operators are used together in different combinations. It's important to understand
difference between:

* `publish().connect()`
* and `publish().refcount()`/`share()`.

### Active and passive observables
Before I go to the heart of the matter, I want to mention the definition of hot
and cold observables.  
For me, hot and cold observables are confusing and somewhat mystic terms
(to make things even more complicated... have you ever heard about warm and cool observables?).

Let's rename "hot" observable to "active sequence" and "cold"
to "passive sequence". These can be defined as follow:

- Active sequences start producing notifications **all the time** regardless of subscriptions
- Passive sequences start producing notifications **on request** (when subscribed to/when asked about it)

Example of a passive sequence might be a network
request which is triggered by subscribing to it. On the other hand, active
sequences are web socket connections, timer events or text values coming from a UITextField.


And that's it. Think about active and passive sequences only, hot/cold/warm/cool
observables are confusing.


### Subscribing many times to the same observable
If you've ever subscribed twice or more to the same observable,
you may have been surprised with the results.

Look at this implementation of a network request:


    let URL = NSURL(string: "http://localhost:4567/example.json")!
    let requestObservable = NSURLSession.sharedSession()
      .rx_data(NSURLRequest(URL: URL))

    requestObservable.subscribeNext {
          print($0)
        }
    requestObservable.subscribeNext {
          print($0)
        }

If you take a look at console logs, you will see two distinct HTTP responses.
Observable performed work twice even if it wasn't our intention.


![Console log](/images/console-log-1.png)


### share() to the rescue
Obviously, that's not what we want, especially with network requests. We can change this behaviour and perform only one network
request by simply adding a `share()` operator.

    let URL = NSURL(string: "http://localhost:4567/example.json")!
    let requestObservable = NSURLSession.sharedSession()
      .rx_data(NSURLRequest(URL: URL))
      .share()

    requestObservable.subscribeNext {
          print($0)
        }
    requestObservable.subscribeNext {
          print($0)
        }

As predicted, only one HTTP response is received.


![Console log](/images/console-log-2.png)


The `share()` operator is basically wrapper for `.publish().refcount()`.


Wait! What are `publish()` and `refcount()` then?
Let's answer this question below.

### publish() and his friend connect()
When `publish()` is used, it transforms an ordinary observable to "connectable
observable". Formal definition says:


> A connectable Observable resembles an ordinary Observable except that it does not begin emitting items when it is subscribed to. **It only emits items when the Connect operator is applied to it**




    print("Creating observable")
    let myObservable = Observable.just(1).publish()

    print("Subscribing")
    myObservable.subscribeNext {
      print("first = \($0)")
    }

    myObservable.subscribeNext {
      print("second = \($0)")
    }

    delay(3) {
      print("Calling connect after 3 seconds")
      myObservable.connect()
    }



    // Output
    Creating observable
    Subscribing
    Calling connect after 3 seconds
    first = 1
    second = 1


In the above example, events are not coming through regardless of subscriptions
made at the same time of observable creation. However, after 3 seconds, we can see
values from both subscriptions.

Simply put, `connect()` activates connectable observable and turns on subscribers.

The fascinating thing is how disposal of subscriptions is handled.
Take a look at this code:


    print("Starting at 0 seconds")
    let myObservable = Observable<Int>.interval(1, scheduler: MainScheduler.instance).publish()
    myObservable.connect()

    let mySubscription = observable.subscribeNext {
      print("Next: \($0)")
    }

    delay(3) {
      print("Disposing at 3 seconds")
      mySubscription.dispose()
    }

    delay(6) {
      print("Subscribing again at 6 seconds")
      myObservable.subscribeNext {
        print("Next: \($0)")
      }
    }


Output:


![Publish Connect](/images/publish-connect.gif)

Even if all subscriptions are disposed, observable still lives and produces events
under the hood. It behaves like active sequence. Now let's compare that to
`.publish().refcount()`.


### The difference between .publish().connect() and .publish().refcount()
You can think of `refcount()` as magical system which handles disposal
of subscriptions for you. `refcount()` calls `connect()` automatically when first observer subscribes so there is no need for doing it manually.


      print("Starting at 0 seconds")
        let myObservable = Observable<Int>.interval(1, scheduler: MainScheduler.instance).publish().refCount()

        let mySubscription = observable.subscribeNext {
          print("Next: \($0)")
        }

        delay(3) {
          print("Disposing at 3 seconds")
          mySubscription.dispose()
        }

        delay(6) {
          print("Subscribing again at 6 seconds")
          myObservable.subscribeNext {
            print("Next: \($0)")
          }
        }

Output:


![Publish Refcount](/images/publish-refcount.gif)


**Notice one thing: When we subscribe again, observable produces brand new elements.**

### Conclusion
Can you see the difference now? `publish().connect()` and `publish().refcount()` (or `share()` as a shortcut)
manage a disposal of observables differently.


When you're using the `publish().connect()`, you have to dispose your observable manually.
Otherwise, it acts like active sequence and produces notifications all the time.


On the other hand, `publish().refcount()`/`share()` keeps track of how many other observers subscribe to observable
and does not disconnect from the observable until the last observer has done so.
In other words, when subscriptions counter drops down to zero, observable is "killed" and does not produce any elements.


It's also worth to check [connect()](http://reactivex.io/documentation/operators/connect.html) and [refcount()](http://reactivex.io/documentation/operators/refcount.html) marbles.


If any of the subject matter remains unclear, please let me know in comments!


*Thanks to [finneycanhelp](https://twitter.com/finneycanhelp) and [thesunshinejr](https://twitter.com/thesunshinejr) for proof-reading this post and suggesting improvements.*
