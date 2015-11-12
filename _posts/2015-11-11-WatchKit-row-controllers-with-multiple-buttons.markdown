---
layout: post
title:  "WatchKit: Row controllers with multiple buttons"
date:   2015-11-11 20:19:00
categories: swift
comments: true
---

Over the last few days, I've been busy working with WatchKit. It was my first
try with Apple Watch (I know, it's November and I'm a little bit late).

If you're new to WatchKit, one of the challenges is figuring out how to
implement table and its rows as it's completely different
from iOS world.

WatchKit doesn't provide `UITableViewDelegate` protocol - it's been
replaced with `table:didSelectRowAtIndex` of the interface controller.
Also, `UITableViewDataSource` protocol doesn't exist - we need to call `setRowTypes:` or
`setNumberOfRows:withRowType:` methods to create rows.

As you can see, it's straightforward process but we hit the wall when we want
to do something more fancy i.e. multiple controls in one single row or multiple
row types.


So here is a common scenario:

We need to create a row with multiple buttons and handle button taps with appropriate
actions. To achieve that, we'll use delegation pattern for identifying which
row and button was clicked.


For this tutorial, we'll build very simple Quiz Watch app to see how it works in action.


![Alt text](/images/01.png)


Let's create simple model which we use for our datasource
(Did you know that world record marathon setter kept his pace faster than the
fastest speed an average treadmill can reach?)

    struct Quiz {
        let question: String
        let firstAnswer: String
        let secondAnser: String

        static func allQuizzes() -> [Quiz] {
            return [
                Quiz(question: "What is the longest river in the world?",
                    firstAnswer: "Nile",
                    secondAnser: "Amazon River"),
                Quiz(question: "What is the world record in marathon?",
                    firstAnswer: "2:02:57",
                    secondAnser: "3:00:59")

            ]
        }
    }

Add `tag` and `delegate` properties in our row controller class. `tag` property
tells us which row has been clicked.

    class QuizRowController: NSObject {
        @IBOutlet var titleLabel: WKInterfaceLabel!
        @IBOutlet var firstAnswerButton: WKInterfaceButton!
        @IBOutlet var secondAnswerButton: WKInterfaceButton!

        var tag: Int?
        var delegate: QuizRowControllerDelegate?
    }

So now, add `protocol`, `IBActions` and `enum` to identify
the position of these buttons.

    protocol QuizRowControllerDelegate: class {
        func quizRowController(_: QuizRowController, didSelectRowWithTag tag: Int, buttonName: ButtonName)
    }

    enum ButtonName {
        case FirstAnwerButton, SecondAnswerButton
    }

    @IBAction func firstAnwerButtonPressed() {
        guard let tag = tag else {
            return
        }
        delegate?.quizRowController(self, didSelectRowWithTag: tag, buttonName: .FirstAnwerButton)
    }

    @IBAction func secondAnwerButtonPressed() {
        guard let tag = tag else {
            return
        }
        delegate?.quizRowController(self, didSelectRowWithTag: tag, buttonName: .SecondAnswerButton)
    }

All we have to do now is configure our rows in `WKInterfaceController`

    override func awakeWithContext(context: AnyObject?) {
        super.awakeWithContext(context)
        configureTableWithData(Quiz.allQuizzes(), rowTypeIdentifier: "QuizRowController")
    }

    func configureTableWithData(data: [Quiz], rowTypeIdentifier: String) {
        table.setNumberOfRows(data.count, withRowType:rowTypeIdentifier)
        for (index, quiz) in data.enumerate() {
            let row = table.rowControllerAtIndex(index) as! QuizRowController
            row.delegate = self
            row.tag = index
            row.titleLabel.setText(quiz.question)
            row.firstAnswerButton.setTitle(quiz.firstAnswer)
            row.secondAnswerButton.setTitle(quiz.secondAnser)
        }
    }


And conform to row controller delegate protocol

    extension InterfaceController: QuizRowControllerDelegate {
        func quizRowController(_: QuizRowController, didSelectRowWithTag tag: Int, buttonName: ButtonName) {
            print(tag)
            print(buttonName)
        }
    }


I've seen some other solutions to the problem (for example, referencing model objects in row
controller classes) but this one doesn't break MVC pattern. That's it!
