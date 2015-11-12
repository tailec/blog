---
layout: post
title:  "NSObject's description method tips"
date:   2015-08-18 16:54:51
categories: blog
comments: true
---
Whenever you create custom object it's useful to override NSObject's description class method.
It returns a string with basic information of objects. However, this output is not very descriptive.

By default this method prints class name and its address in memory but you can generate a nicely formatted string which is easy to read.

Let's say we have simple class describing car:

~~~ objc
@interface Car : NSObject

@property (copy, nonatomic)   NSString *model;
@property (copy, nonatomic)   NSString *make;
@property (strong, nonatomic) NSDate *registrationDate;
@property (assign, nonatomic) int mileage;
@property (assign, nonatomic) double fuelConsumption;

@end
~~~
And now see what input we've got without overriding NSObject's description method which is not very useful:

~~~ objc
2015-01-18 19:52:42.875 cli[1828:43955] <Car: 0x100112110>
Program ended with exit code: 0
~~~

But we can override it:

~~~ objc
- (NSString*)description {
    return [NSString stringWithFormat:@"<%@:%p \n model = %@ \n make = %@ \n registrationDate = %@ \n mileage = %d \n fuelConsumption = %f", [self className], self, self.model, self.make, self.registrationDate, self.mileage, self.fuelConsumption];
}
~~~

And see the result:

~~~ objc
2015-01-18 20:21:33.393 cli[1893:49109] <Car:0x100207ed0
 model = Renault
 make = Megane
 registrationDate = 2015-01-18 20:21:33 +0000
 mileage = 52000
 fuelnConsumption = 10.200000
Program ended with exit code: 0
~~~

It requires a little bit of work with string formatting and if you've got massive object with loads of properties it could be overwhelming. Of course we can take a step futher and use already formatted NSDictionary/NSArray description methods.

~~~ objc
- (NSString*)description {
    return [NSString stringWithFormat:@"<%@:%p %@>",
            [self className],
            self,
            @{ @"model"           : self.model,
               @"make"            : self.make,
               @"registrationDate": self.registrationDate,
               @"mileage"         : @(self.mileage),
               @"fuelConsumption" : @(self.fuelConsumption)
            }];
}
~~~

Which prints out this:

~~~ objc
2015-01-18 20:34:27.995 cli[1902:51134] <Car:0x1001120b0 {
    fuelConsumption = "10.2";
    make = Megane;
    mileage = 52000;
    model = Renault;
    registrationDate = "2015-01-18 20:34:27 +0000";
}>
Program ended with exit code: 0
~~~

This is faster solution but still it's not perfect because we have to do it in every class we create.

You may find it helpful to use a simple category on NSObject which handles that for you. You can find on my [github](https://github.com/tailec/NSObject-Description-category).
