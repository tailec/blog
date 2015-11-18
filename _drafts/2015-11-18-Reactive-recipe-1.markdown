---
layout: post
title:  "Reactive recipe #1"
date:   2015-11-17 22:52:00
categories: swift
comments: true
---

I've been looking into Reactive Programming in general
and I can say one thing: it's MIND-BLOWING concept!

As part of my learning process, I came up with idea of sharing my thoughts and
feelings on Rx. This post is introduction to series of blog posts about real-world
iOS examples using RxSwift.


### Why RxSwift?
I picked up RxSwift as it's very well documented with lots of sample code snippets
and great community (go to theirs slack channel and see yourself).
There is also [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
which is fully grown project but I like the idea behind RxSwift - this is part of
[reactivex](http://reactivex.io/). It means that same base reactive programming
concepts apply to different languages (Java, C#, JavaScript etc.).

### Who is this for?
Those recipes are for beginners like me who already know the basics of reactive
programming but are confused how to use them in real-world scenarios.

If you are new to Rx, I suggest you read these in following order:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
Yes, it's JavaScript but syntax is not that important
- [Why Reactive(Cocoa)?](http://www.sprynthesis.com/2014/06/15/why-reactivecocoa/)
Yes, it's ReactiveCocoa not RxSwift
- [ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)
ReactiveCocoa again!
- [Functional Reactive Programming on iOS](https://leanpub.com/iosfrp/)
Book written by Ash Furrow. You guessed! It's about ReactiveCocoa
- **[More links](http://tailec.com/blog/startwith-reactivecocoa-and-mvvm)**

Unfortunately, there aren't many tutorials/guides for RxSwift :(

### Dive in
Let's start with the basic example - sign in functionality using
[Parse](www.parse.com) backend.


So to get started, we'll create single page-based app with only one view
controller. This view controller will contain username text field, password
text field and 'Sign in' button.

![Reactive recipe #1](/images/02.png)

    import UIKit
    import RxSwift
    import RxCocoa
    import Parse

    class ViewController: UIViewController {

        @IBOutlet weak var usernameTextField: UITextField!
        @IBOutlet weak var passwordTextField: UITextField!
        @IBOutlet weak var loginButton: UIButton!

        override func viewDidLoad() {
            super.viewDidLoad()
        }
    }

First of all, we need to validate user input. Create two variables of `Observable`
type in `viewDidLoad()` method. They return `bool` value depending on validation logic
(username or password must be longer than 5 characters).

    let validUsernameObservable = usernameTextField.rx_text.map { username in
       username.characters.count > 5
    }.distinctUntilChanged()

    let validPasswordObservable = passwordTextField.rx_text.map { password in
       password.characters.count > 5
    }.distinctUntilChanged()

Let's test above the code and try to type something in username text field:

    validUsernameObservable.subscribeNext { value in
        print(value)
    }
    // prints:
    // false
    // true
    // false
    // true

Or test it visually:

    validUsernameObservable.map { value in
        value ? UIColor.greenColor() : UIColor.whiteColor()
    }
    .subscribeNext { color in
        self.usernameTextField.backgroundColor = color
    }

That's it, the username text field is highlighted green when username is valid.

So far so good. What if we disable **Sign in** button when text fields are invalid?
It's not that hard if you know basic Rx operators. `bindTo` extension is quite
new thing in RxSwift - read more [here](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Examples.md#simple-ui-bindings).

    combineLatest(validUsernameObservable, validPasswordObservable) { usernameValid, passwordValid in
        return usernameValid && passwordValid
    }.bindTo(loginButton.rx_enabled)

Now, we just need to wrap Parse network request method into `Observable`.

    func parseLogin() -> Observable<String> {
         return create { observer in
             PFUser.logInWithUsernameInBackground(self.usernameTextField.text!, password: self.passwordTextField.text!) { user, error in
                 observer.onNext(user?.username ?? "Try again!")
                 observer.onCompleted()
             }
             return NopDisposable.instance
         }
     }

In order to sign in, we subscribe to `rx_tap`, do some magic with
`flatMap` and show alert on the screen.


`flatMap` was hard one for me to understand. It's used because we want
to return `String` which is value, not `<Observable<String>` which is `Observable`
(give it a go and try using `map` instead of `flatMap` - it won't even compile).
By using `map`, we could create something what I call *inner observable* (an `Observable`
within another `Observable`).


    loginButton.rx_tap.flatMap {
       parseLogin()
    }
    .subscribeNext {
       let alert = UIAlertController(title: $0, message: nil, preferredStyle: UIAlertControllerStyle.Alert)
       alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
       self.presentViewController(alert, animated: true, completion: nil)
    }


If you got lost along the way, you can look at my code in **[GitHub](https://github.com/tailec/reactive-recipes)**.
