# Introduction #

Google Toolbox For Mac (GTM) is extensively unit tested. As of this writing we have about 91% code coverage with our unit tests. We expect any submissions to GTM to come with a complete unit test suite, testing as much functionality as is reasonably possible. GTM has several enhancement to the standard SenTestingKit allowing you to do UI unit testing, automated binding unit testing, log tracking, and unit testing on the iPhone, as well as tools for doing static and dynamic testing of your code.


# GTM Unit Testing Basics #
At its base GTM uses the SenTestingKit framework supplied with the Apple Development Tools. We have added significant enhancements though to aid in getting better code coverage, and faster issue resolution. The key files for the basic enhancements is GTMSenTestCase.m/.h.

Please use the following macros as appropriate instead of falling back on `STAssertTrue`/`STAssertFalse`. These macros will display far more information when an assert fails making debugging easier. For documentation on these asserts please see GTMSenTestCase.h

```
STAssertNoErr
STAssertErr
STAssertNotNULL
STAssertNULL
STAssertNotEquals
STAssertNotEqualObjects
STAssertOperation
STAssertGreaterThan
STAssertGreaterThanOrEqual
STAssertLessThan
STAssertLessThanOrEqual
STAssertEqualStrings
STAssertNotEqualStrings
STAssertEqualCStrings
STAssertNotEqualCStrings
```

All GTM unit tests should inherit from `GTMTestCase` instead of `SenTestCase`. `GTMTestCase` enables the ability to use log tracking and is required on iPhone for iPhone unit tests to work correctly. `GTMTestCase` also enhances `SenTestCase's` ability to work with more complex class hierarchies. `SenTestCase` is not designed out of the box to handle an abstract class hierarchy descending from it with some concrete subclasses.  In some cases we want all the "concrete" subclasses of an abstract subclass of `SenTestCase` to run a test, but we don't want that test to be run against an instance of an abstract subclass itself. You can specify that a`GTMTestCase` subclass is an abstract test case by including "AbstractTest" in the class name (e.g. `FooAbstractTestCase`).

# GTM Log Tracking #
When running unit tests we want to exercise all the code we possibly can. Hopefully this means entering into areas of the code  where we have `_GTMDevLog` calls to indicate that this may not be expected in the normal flow of execution. This means when running unit tests that we will get a screen full of log messages making it easy to miss an unexpected log that is indicating that something unexpected has occurred. GTM Log Tracking allows you to specify what logs you expect to fire during the execution of a unit test, and will error if you hit an unexpected log or if a log you expect to fire does not fire. Log tracking is optional and is not required to be turned on.

To enable GTM log tracking add GTMUnitTestDevLog.m/.h to your target and redefine `_GTMDevLog` to call `_GTMUnittestDevLog`. We have done this in the GTM Framework in GTM\_Prefix.pch.

Now that log tracking is enabled any logs that fire in the execution of a unit test will cause an error. We now need to let the log tracking system know which logs are expected. To do this use the appropriate methods of GTMUnitTestDevLog as described in GTMUnitTestDevLog.h.

The `-expect...` methods allow you to specify an extra string that you are expecting, or a regexp pattern to match against. For example:

```
[GTMUnitTestDevLog expectPattern:@"Object ptr 0x[0-9A-F]* is illegal"];
STAssertFalse([foo insertObject:obj], @"Should fail inserting invalid object");
```

The log tracking system will now complain if it gets a log that doesn't match this regular expression, or if this expected log is never fired.

Important notes for everything to work well:
  * Your test case classes must inherit from `GTMTestCase` for full functionality. Explicitly the ability to detect if an expected log is never fired.
  * You must use `_GTMDevLog` calls to log. We will not catch `NSLog`, `fprintf`, or any other logging system unless you juryrig support yourself.

# GTM UI Unit Testing #
GTM has extensive support for user interface unit tests. It supports testing both the imaging and/or internal state of almost all of the standard Cocoa/UIKit UI objects, and makes it easy for you to extend this support to your own UI objects. The key files for this support are GTMNSObject+UnitTesting.m/.h, and depending on your needs you will probably require one or more of: GTMAppKit+UnitTesting, GTMCALayer+UnitTesting or GTMUIKit+UnitTesting.

To do imaging tests use the `GTMAssertObjectImageEqualToImageNamed` macro. This compares the drawing of an object, and its subviews, to a master image file stored in the test executable's bundle. If the master image file does not exist yet, it will be created and placed on the Desktop and then easily added to the project. Master image files can be specialized on OS Version and architecture in case your image draws differently on Tiger vs Leopard, or ppc vs i386. If a unit test fails it will place a copy of the new image on the desktop as well as an image of a diff of the old vs new images. If your UI object is a subclass of NSView it will most likely unit test correctly with no modifications necessary. If your class is not a subclass of NSView but does drawing you may consider using the `GTMAssertDrawingEqualToImageNamed` macro defined in GTMAppKit+UnitTesting.h. This allows you to easily test drawing routines that aren't themselves a view. See GTMNSBezierPath+CGPathTest.m for an example of using `GTMAssertDrawingEqualToImageNamed`.

For testing onscreen drawing it is often important to have a consistent environment to test against. GTMUnitTestingUtilities contains routines for setting up a consistent UI state, such as standardizing scroll bar types, selection colors, color spaces etc. It will automatically revert back to the original state when the unit tests have finished executing. See GTMUIUnitTestingHarness/main.m for an example of how to use `[GTMUnitTestingUtilities setUpForUIUnitTestsIfBeingTested]`. Note that there are other cases of state dependent drawing (such as date controls) that you will need to be careful with when doing image based testing.

To test the internal state of an object use the `GTMAssertObjectStateEqualToStateNamed` macro. This compares the internal state of an object, and its subviews, to a master state file stored in the test executable's bundle. If the master state file does not exist yet, it will be created and placed on the Desktop and then easily added to the project. Master state files can be specialized on OS Version and architecture in case your state is different on Tiger vs Leopard, or ppc vs i386. If a unit test fails it will place a copy of the new state on the desktop. State files are standard plist format. If you want to extend the state unit testing to your own UI objects you can either implement
```
- (void)gtm_unitTestEncoderWillEncode:(id)sender inCoder:(NSCoder*)inCoder;
```
for your object in a category that you include in your unit test bundle, or listen for the `GTMUnitTestingEncodedObjectNotification` notification and respond to it appropriately.

See GTMAppKit+UnitTesting.m for examples of how to support both state and imaging tests.

Important notes for everything to work well:
  * For imaging, make sure that your testing environment is set up correctly.
  * Make sure to test on both Leopard and Tiger (and any other OS as appropriate) as drawing is often subtly different between both OS releases and hardware platforms.

# GTM Binding Unit Testing #
GTM binding unit testing allows you to automatically test any classes you have that support NSBinding, saving you from having to write a whole pile of set/get test code to test them yourself. The files you will require are GTMNSObject+BindingUnitTest.m/.h. To have your bindings tested, your object will need to implement
```
- (NSArray *)exposedBindings
```
and
```
- (Class)valueClassForBinding:(NSString *)binding
```
By default calling `GTMTestExposedBindings` on your object that supports bindings will test all of bindings exposed by `-exposedBindings` with several interesting edge cases. For more extensive testing you may possibly want to implement `-gtm_unitTestExposedBindingsToIgnore` and `-gtm_unitTestExposedBindingsTestValues`. See descriptions of those methods in GTMNSObject+BindingUnitTest.h for details.

# GTM HTTP Unit Testing #
For testing objects that interact with remote servers we supply `GTMHTTPServer`. `GTMHTTPServer` is probably not what you want to use for an industrial stength webserver, but it works great for easily testing protocols without requiring firing up a webserver in a separate process. See GTMHTTPFetcherTest.m for an example of how to use it effectively.

# GTM iPhone Unit Testing #
GTM supports all of the features described above on the iPhone if they are applicable. Specifically we support UI unit testing, log tracking and all of the standard (and our additional) STMacros on the iPhone in both Simulator and Device mode. GTM does not support binding unit testing on the iPhone as NSBinding is not supported on the iPhone. To have your tests executed on the phone, make sure your iPhone application's delegate is `GTMIPhoneUnitTestDelegate`, or calls `[GTMIPhoneUnitTestDelegate runTests]` when you want your tests to execute. All of your tests should execute as you would expect. Please see [iPhoneUnitTesting](http://code.google.com/p/google-toolbox-for-mac/wiki/iPhoneUnitTesting) for a tutorial on setting up testing for the iPhone.

# GTM Code Coverage Analysis #
GTM has code coverage targets that build with full gcov settings on. We then use [CoverStory](http://code.google.com/p/coverstory) to look at our code coverage. In some small isolated cases it is impossible to reach blocks of code, and in these cases we allow the use of the [non-feasible code comments](http://code.google.com/p/coverstory/wiki/NonFeasibleCode). Please be prepared to defend your usage of them. In all cases we hope to have 100% code coverage, and this should be attainable with good design, thorough test cases, and minimal usage of [non-feasible code comments](http://code.google.com/p/coverstory/wiki/NonFeasibleCode).

# Other Code Verification Features and Utilities #
GTM has several other features for statically and dynamically verifying code correctness, especially in debug builds. Please make sure to use these in any code that you are contributing back to GTM.

## GTMDebugSelectorValidation ##
For objects that expect to call selectors on an anonymous object, for example a delegate object which partially conforms to a informal protocol, we supply `GTMAssertSelectorNilOrImplementedWithArguments` and `GTMAssertSelectorNilOrImplementedWithReturnTypeAndArguments` in GTMDebugSelectorValidation.h. When a delegate is assigned to your object, you can use these macros on each of the methods that you expect the delegate to implement to make sure that the selectors have the correct arguments/return types. These are not compiled into release builds.

## GTMMethodCheck ##
When using categories, it can be very easy to forget to include the implementation of a category. Let's say you had a class foo that depended on method bar of class baz, and method bar was implemented as a member of a category on baz. Xcode will happily link your program without you actually having a definition for bar included in your project. The `GTM_METHOD_CHECK` macro checks to sure baz has a definition just before main is called. This works for both dynamic libraries, and executables.

Example of usage:
```
@implementation foo
  GTM_METHOD_CHECK(baz, bar)
@end
```

Classes (or one of their superclasses) being checked must conform to the  NSObject protocol. We will check this, and spit out a warning if a class does not conform to NSObject. `GTM_METHOD_CHECK` is defined in GTMMethodCheck.h and requires you to link in GTMMethodCheck.m as well. None of this code is compiled into release builds. See GTMNSAppleScript+Handler.m for examples of using `GTM_METHOD_CHECK`.

## GTMTypeCasting ##
[GTMTypeCasting.h](http://code.google.com/p/google-toolbox-for-mac/source/browse/trunk/DebugUtils/GTMTypeCasting.h) contains some macros for making down-casting safer in Objective C.
They are loosely based on the same cast types with similar names in C++.
A typical usage would look like this:
```
 Bar* b = [[Bar alloc] init];
 Foo* a = GTM_STATIC_CAST(Foo, b);
```

Note that it's `GTM_STATIC_CAST(Foo, b)` and not `GTM_STATIC_CAST(Foo*, b)`.

`GTM_STATIC_CAST` runs only in debug mode, and will assert if and only if:
  * object is non nil
  * `[object isKindOfClass:[cls class]]` returns nil

otherwise it returns object.

`GTM_DYNAMIC_CAST` runs in both debug and release and will return nil if
  * object is nil
  * `[object isKindOfClass:[cls class]]` returns nil

otherwise it returns object.

Good places to use `GTM_STATIC_CAST` is anywhere that you may normally use a straight C cast to change one Objective C type to another. Another less visible place is when converting an `id` to a more specific Objective C type. A classic place that this occurs is when getting the `object` from a notification.

```
- (void)myNotificationHandler:(NSNotification *)notification {
  MyType *foo = GTM_STATIC_CAST(MyType, [notification object]);
  ...
```

This will help you quickly trap cases where the type of notification object has changed.

## RunMacOSUnitTests ##
The unit tests in GTM use an enhanced script for running their unit tests that "encourages" unit tests to fail by setting a variety of environment variables such as zombies, MallocDebug etc. All unit test targets should use RunMacOSUnitTests to run their unit tests as part of their build.  See RunMacOSUnitTests.sh for details.

## Strict Compiler Warning Settings ##
We are slowly but surely tightening down on the warnings settings in GTM. These settings will be changed in the appropriate xcconfig files. If your code stops compiling because of a new release of GTM it will likely be because of these warning changes. We realize in some cases that it is a pain to "work around" a warning, but in general if a warning is flying there is usually an easy way to fix it and in the end it leads to better code safety.

## Compile Time Asserts ##
For checking things at compile time we have `_GTMCompileAssert`. This allows us to verify assumptions about things that the compiler knows about at compile time such as sizes of structures. Please use this whenever applicable. `_GTMCompileAssert` is defined in GTMDefines.h and you can see examples of it's use in GTMGeometryUtils.h.

# Conclusion #
We are always interested in other methods of verifying our code correctness. Please share any ideas you have with us, and we will be happy to introduce good ones to the GTM codebase. If you see places we have missed using any of our internal verification features, please log bugs, or even better send us patches. If you would like us to expand on any of the above documentation please let us know. Issues can be logged here. Please note that all of the above techniques are opensource, and feel free to use them in your own code to improve it. QA is an expensive commodity, and hopefully some of the code/techniques above will allow you to develop more robust code.