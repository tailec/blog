---
layout: post
title:  "MVVM with RxSwift experiment"
date:   2016-02-13 14:00:34
categories: swift
comments: true
---

Someone told me recently,

_"In read world, view models should work even if you remove user input"_

and it gave me an idea of transferring my view models from iOS app to Mac app.

The apps are performing github repositories search based on query provided by
user.

They have only one view model, one model object representing *Repository* and
networking layer ([Moya](https://github.com/Moya/Moya)) which are completely
the same in iOS and Mac app.

The component which changes is a view controller. It's responsible for binding
inputs and outputs of view model and view controller:


~~~ swift
// Input
var searchText = Variable("")
// Output
let searchResults: Driver<[Repo]>
~~~


iOS view controller is built using *UISearchBar* and *UITableView*, whilst
Mac view controller uses *NSSearchField* and *NSTableView*.


This is very simple example without scene transitions etc.,
although it works really well.

The code can be found at [github](https://github.com/tailec/mvvm-experiment).
