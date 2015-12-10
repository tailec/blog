---
layout: post
title:  "Reactive recipe #2"
date:   2015-12-10 20:21:34
categories: swift
comments: true
---
This recipe is divided into sections which are answers to the questions I asked myself while
developing my first MVVM pattern with usage of RxSwift.


I don't want to go in details how MVVM works as there are many fantastic
articles which describe this pattern. I want to focus on implementation specifics in RxSwift.

If you have never come across MVVM before, I strongly recommend these resources
in following order:

- [Introduction to MVVM by Ash Furrow](https://www.objc.io/issues/13-architecture/mvvm/)
- [ReactiveCocoa and MVVM, an Introduction by Bob Spryn](http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/)

As a result of my observations, I've curated a list of principles that I try to follow:

View Controllers:

- are responsible for controlling views (setting background colors, AutoLayout
  constraints etc.)
- tell view models when something happens (user taps a button or writes something
  in text field)
- listen to view model (network request completes so reload tableView)
- never reference models

View Models:

- perform all the business logic (fire-off network requests, fetch Core Data objects,
  validate input provided by user etc.)
- don't import UIKit
- deliver necessary data to view controllers to display itself
- update and listen to changes in models

Models and views are pretty the same as in standard MVC pattern.


For purpose of this recipe, I've created yet another ToDoList app which can be
found on my [GitHub](https://github.com/tailec/reactive-recipes/tree/master/reactive-recipe-2).


### What is the best place to initialize view models? ###
I've seen different approaches of injecting view models to view controllers:

- store them as `lazy var` inside view controllers
- initialize them in view controllers' `viewDidLoad`
- create view models before initializing view controllers

All of them have advantages and drawbacks but I prefer third option. Why? Because
it can be injected as part of the initializer which is most logical for me.


    let mainViewModel = MainViewModel(coreDataStack: coreDataStack)
    let mainViewController = MainViewController(viewModel: mainViewModel)
    let navigationController = UINavigationController(rootViewController: mainViewController)

### What are inputs and outputs of view model? ###
View models have inputs and outputs. It simply means that they receive some
data from view controller (user taps the button) or model (core data context has changed),
transform this data and pass to view controller.

    // Input
    var searchTextObservable = PublishSubject<String>()

    // Output
    var contentChangesObservable: Observable<[Item]>
    var titleObservable: Observable<String>

    // Private
    private var coreDataStack: CoreDataStack
    private let disposeBag = DisposeBag()

Outputs are usually `Observables` and inputs are some sort of `Subject` (`Variable`, `BehaviourSubject`
or `PublishSubject`).

### Bindings between view controller and view model ###
Inputs are defined as Subjects because they are Observables and Observers at the
same time. It allows us to bind them (`bindTo`) and subscribe to them in view
models:

    // ViewController
    _ = searchBar.rx_text.bindTo(viewModel.searchTextObservable)

In that case, whenever user writes in search bar, view model receives this text:

    // ViewModel
    _ = searchTextObservable
      .subscribeNext { text in
        print(text)
        // prints whatever user enters in view model
      }

    titleObservable = searchTextObservable
      .map { text in
        "Search: " + text
      }

`titleObservable` is output observable so view controller should subscribe to it:

    // ViewController
    viewModel.titleObservable
    .subscribeNext { [unowned self] textTitle in
        self.title = textTitle
    }

### How to tell view model when view controller is ready to present UI to the user? ###
Ideally, view model should be active when all the views in
view controller are ready for work.

It can be achieved by setting `active` property on the view model
to `true/false` in `viewWillAppear/viewWillDisappear`:

    // View Controller
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        viewModel.active = true
    }

    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)
        viewModel.active = false
    }

    // View model
    init() {
      super.init()
      // didBecomeActive observable emits event when active == true
      didBecomeActive.subscribeNext {

          print("View model is ready to perform work")
      }
    }


`didBecomeActive` and friends are available in [RxViewModel](https://github.com/RxSwiftCommunity/RxViewModel)

### How to handle navigation? ###
In example project, I've created function in view model which returns another view model.


    // Main View Model
    func addViewModel() -> AddViewModel {
        return AddViewModel(coreDataStack: coreDataStack)
    }

    // Main View Controller
    let addViewController = AddViewController(viewModel: self.viewModel.addViewModel())
    let navigationController = UINavigationController(rootViewController: addViewController)
    self.presentViewController(navigationController, animated: true, completion: nil)

I like this solution because it doesn't violate MVVM (view controller still doesn't
know about model layer).




In next recipe, I'll try to write some units tests for RxSwift+MVVM.


Example project can be found on [GitHub](https://github.com/tailec/reactive-recipes/tree/master/reactive-recipe-2).
