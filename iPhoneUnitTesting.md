# Introduction #

For a long time, Apple didn't provide a way to do iOS unit testing, so Google Toolbox For Mac helped fill that gap by providing all the support to write tests with SenTest just like you could for OS X.  But as of Xcode 4.x, Apple has supported unit testing iOS and taken it even further than before with Xcode integration for controlling what tests are run, debugger support, etc.  Google Toolbox For Mac on iOS now supports both the older Google Toolbox For Mac only way (building a App and GTM does everything) and working in parallel with Apple's support (like Google Toolbox For Mac does for OS X testing; using the Apple provided testing frameworks, just adding additional functionality).

<br />

---

# GTM Unit Testing with Apple's support/Xcode integration #

This is now the suggested way to using Google Toolbox For Mac for unit testing on iOS.

## Basic Project Setup ##

Start by follow Apple's [UnitTesting documentation](http://developer.apple.com/library/ios/#documentation/DeveloperTools/Conceptual/UnitTesting/00-About_Unit_Testing/about.html).  Then add `GTM_IPHONE_USE_SENTEST=1` to your `GCC_PREPROCESSOR_DEFINITIONS` settings of your testing target(s). Lastly, add `google-toolbox-for-mac/UnitTesting/GTMSenTestCase.m` and `google-toolbox-for-mac/UnitTesting/GTMSenTestCase.h` to your testing target(s).

That's it, you'll get the added support the GTM provides to test cases using the stock Apple supported SenTest.

## Supporting Command Line Building/Testing ##

Apple describes two types of tests, Logic and Application tests.  The Application tests work by hosting the test bundle inside of the Application.  As of Xcode 4.6, Apple still doesn't support running these type of tests in command line builds.  Google Toolbox for Mac provides a replacement script (RuniOSUnitTestsUnderSimulator.sh) that can be used in place of the one Xcode provides that is able to launch these tests from the command line.  Simply update the Script phase on your testing target(s) to use it instead of the one bundled with Xcode.

<br />

_That's it, ignore the next section, as it is for the other/older method of testing._

<br />

---

# Older: GTM Unit Testing _without_ Apple's support/Xcode integration #

This is a quick tutorial on the older way of doing iOS unit testing using the facilities in the Google Toolbox For Mac. Please send mail to [the group](http://groups.google.com/group/google-toolbox-for-mac) if any clarification is required.

## Basic Project Setup ##
Hopefully your project already has an application target for building something for the iPhone.
  1. Create a new iPhone Target of type "Cocoa Touch Application" via "Project Menu > New Target...". Choose a name that makes some sense such as "Unit Test". Be sure to use a "Cocoa Touch Application" target as opposed to a "Cocoa Application" target or a "Cocoa Touch Static Library" target or "Cocoa Touch Unit Test Bundle".
  1. Add `google-toolbox-for-mac/UnitTesting/GTMIPhoneUnitTestMain.m` to your target
  1. Add `google-toolbox-for-mac/UnitTesting/GTMIPhoneUnitTestDelegate.m` to your target
  1. Add `google-toolbox-for-mac/UnitTesting/GTMIPhoneUnitTestDelegate.h` to your project
  1. Add `google-toolbox-for-mac/UnitTesting/GTMSenTestCase.m` to your target
  1. Add `google-toolbox-for-mac/UnitTesting/GTMSenTestCase.h` to your project
  1. Add `google-toolbox-for-mac/UnitTesting/GTMUnitTestDevLog.m` to your target
  1. Add `google-toolbox-for-mac/UnitTesting/GTMUnitTestDevLog.h` to your project
  1. Add `google-toolbox-for-mac/Foundation/GTMObjC2Runtime.m` to your target
  1. Add `google-toolbox-for-mac/Foundation/GTMObjC2Runtime.h` to your project
  1. Add `google-toolbox-for-mac/Foundation/GTMRegex.m` to your target
  1. Add `google-toolbox-for-mac/Foundation/GTMRegex.h` to your project
  1. Add `google-toolbox-for-mac/GTMDefines.h` to your project
  1. Add a new 'run script' build phase as the last step of your target build via "Project Menu > New Build Phase > New Run Script Build Phase", and dragging it to the end of the build steps if needed.
  1. Edit your Run Script Build Phase by double clicking it, and set the shell to "/bin/sh" and the script to `"PATH_TO_GTM/UnitTesting/SCRIPT_NAME"`, where PATH\_TO\_GTM is the path to your local version of google-toolbox-for-mac and SCRIPT\_NAME should be RunIPhoneUnitTest.sh if you are using Xcode 4.4 or lower and RuniOSUnitTestsUnderSimulator.sh if you are using Xcode 4.5 or higher.
  1. Xcode's new application stationery generates an Info.plist that specifies a Main nib file base name. Open the new unit test's .plist file and remove that line. If you don't do this, RunIPhoneUnitTest.sh will crash when UIApplication throws an exception trying to load an inappropriate or non-existent nib file.
  1. Build! Note that if you choose build and go you will see your unit tests executed twice, once as part of the build script, and once being run

Your target should now build cleanly, and if you check the build log you should see something like:
"Executed 0 tests, with 0 failures (0 unexpected) in 0.001 (0.001) seconds" at the end.

## Trouble Shooting ##
  * Make sure you are not linked against the `SenTestingKit.framework`.
  * RunIPhoneUnitTest.sh vs. RuniOSUnitTestsUnderSimulator.sh: The scripts will attempt to warn you if you are using the wrong one, Xcode 4.5 changed how the simulator can be launched, so RunIPhoneUnitTest.sh no longer works.  RuniOSUnitTestsUnderSimulator.sh uses a new method that is closer to what Xcode does when launching applications under the simulator.

## Creating a unit test ##
  1. Add the source you want to test to your target. For example if you want to test class Foo, make sure to add Foo.h and Foo.m to your target.
  1. Add a new unit test file to your target via "File > New File" and choose "Objective-C test case class" from the "Mac OS X" Cocoa category. Call it `FooTest.m`, or follow whatever convention your project has for naming test classes.
  1. In `FooTest.h`, change `#import <SenTestingKit/SenTestingKit.h>` to `#import "GTMSenTestCase.h"`
  1. Also set up your class so that it inherits from GTMTestCase (_not_ GTMSenTestCase) instead of `SenTestCase` <br> <code> @interface MyTestCase : GTMTestCase { ... } </code>
<ol><li>Add test cases as you normally would. See <a href='http://developer.apple.com/tools/unittest.html'>Apple's Documentation</a> for a good tutorial on how to test in Objective C. The key is that your test case methods are declared <code>- (void)testBar</code>. The name must start with "test" and they must return nothing and have no arguments.</li></ol>

Now when you build your target you should now see test cases executing.  You can repeat this process creating additional source files for each class you want to write Unittests for.<br>
<br>
<h2>Debugging</h2>
You can debug your unit test executable the exact same way you would unit test any executable. You shouldn't have to set up anything. Note that you may see cases where the unit test build fails, but it doesn't fail when you are debugging. When the unit test build is running several flags are turned on to encourage failures such as <code>MallocScribble</code>, <code>NSAutoreleaseFreedObjectCheckEnabled</code> etc. You may want to look in <code>RunIPhoneUnitTest.sh</code> and see what is being enabled for you.<br>
<br>
<br />
<hr />
<h1>Notes</h1>

These apply to both forms of UnitTesting that GTM supports.<br>
<br>
<ul><li>We find that having a .h for tests to be mostly useless, and tend to just the interfaces for the tests in with the implementation so I only have one file to worry about.<br>
</li><li>Make sure to check out the extra ST macros that we have added in <code>GTMSenTestCase.h</code> that go above and beyond the set included with the default OCUnit.<br>
<ul><li>STAssertNoErr(a1, description, ...), STAssertErr(a1, a2, description, ...)<br>
</li><li>STAssertNotNULL(a1, description, ...), STAssertNULL(a1, description, ...)<br>
</li><li>STAssertNotEquals(a1, a2, description, ...), STAssertNotEqualObjects(a1, a2, desc, ...)<br>
</li><li>STAssertEqualObjects(a1, a2, description, ...), STAssertEquals(a1, a2, description, ...), STAssertEqualsWithAccuracy(a1, a2, accuracy, description, ...)<br>
</li><li>STAssertOperation(a1, a2, op, description, ...), STAssertGreaterThan(a1, a2, description, ...), STAssertLessThan(a1, a2, description, ...), STAssertLessThanOrEqual(a1, a2, description, ...)<br>
</li><li>STAssertEqualStrings(a1, a2, description, ...), STAssertNotEqualStrings(a1, a2, description, ...), STAssertEqualCStrings(a1, a2, description, ...), STAssertNotEqualCStrings(a1, a2, description, ...)<br>
</li><li>STAssertTrueNoThrow(expr, description, ...), STAssertFalseNoThrow(expr, description, ...), STAssertThrows(expr, description, ...), STAssertThrowsSpecific(expr, specificException, description, ...), STAssertThrowsSpecificNamed(expr, specificException, aName, description, ...), STAssertNoThrow(expr, description, ...), STAssertNoThrowSpecific(expr, specificException, description, ...), STAssertNoThrowSpecificNamed(expr, specificException, aName, description, ...)<br>
</li></ul></li><li>You can't run the build script while the iPhone simulator is running. The build script does attempt to kill it off before it runs, but if you see <code>Couldn't register PurpleSystemEventPort with the bootstrap server. Error: unknown error code. This generally means that another instance of this process was already running or is hung in the debugger.</code> or <code>Abort trap "$TARGET_BUILD_DIR/$EXECUTABLE_PAT" -RegisterForSystemEvents</code> you probably need to figure out why the simulator (or another iPhone process) is already running.  The exact error has changed with different versions of the iPhone SDK.</li></ul>

<br />
<hr />
<h1>Advanced Stuff</h1>

<i>NOTE:</i> Some of these only apply to the older form of GTM UnitTesting support.  Check the script for launching UnitTests for the latest information on what is supported where.<br>
<br>
<h2>Unit test Logging</h2>
When Unittesting is done correctly, you often have a lot of log messages logging because you are testing edge cases that you may not expect to hit in the real world very often. It's nice to be able to verify that the log messages you are receiving as you run your tests are the ones that you expect to receive. You can do this in GTM by enabling unit test logging.<br>
<br>
<ol><li>Assuming you are using <code>_GTMDevLog</code> to do your logging,<code>#define _GTMDevLog _GTMUnittestDevLog</code> somewhere, either in target settings, or in your prefix. If you are using NSLog you can just define it to be <code>_GTMUnittestDevLog</code>.<br>
</li><li>Add <code>google-toolbox-for-mac/DebugUtils/GTMDevLog.m</code> to your target.<br>
</li><li>Add <code>google-toolbox-for-mac/UnitTesting/GTMUnitTestDevLog.m</code> to your target.<br>
</li><li>Add <code>google-toolbox-for-mac/Foundation/GTMRegex.m</code> to your target.<br>
</li><li>You may also need to add some headers depending on your search paths</li></ol>

Now when you build all of the logging that you do via your unit tests will get checked to make sure that it conforms with your expectations. You set up these expectations before running your tests using <code>[GTMUnitTestDevLog expect*]</code> methods. See <code>GTMUnitTestDevLog.h</code> for more info.<br>
<br>
<h2>UI and State Testing</h2>
GTM can also help you test your UI's representation and state.<br>
<br>
<ol><li>Add <code>google-toolbox-for-mac/UnitTesting/GTMNSObject+UnitTesting.m</code> to your target.<br>
</li><li>Add <code>google-toolbox-for-mac/UnitTesting/GTMUIKit+UnitTesting.m</code> to your target.<br>
</li><li>Add <code>google-toolbox-for-mac/UnitTesting/GTMCALayer+UnitTesting.m</code> to your target.<br>
</li><li>Add <code>google-toolbox-for-mac/Foundation/GTMSystemVersion.m</code> to your target.<br>
</li><li>Add the <code>CoreGraphics</code> and <code>QuartzCore</code> frameworks to your target.<br>
</li><li>You may also need to add some headers depending on your search paths</li></ol>

Check out <code>UnitTesting/GTMUIKit+UnitTestingTest.m</code> for examples of using<code> UIUnitTesting</code> on the iPhone.<br>
<br>
For more information on some of this, check out CodeVerificationAndUnitTesting. Hope this helps.<br>
<br>
<h2>Unit Test Environment Variables</h2>
To encourage "bad behavior" by the code being tested, the <code>RunIPhoneUnitTest.sh</code> script sets a variety of environment variables. If you are wondering why the unit tests fail when you are building, but don't fail when you are running, it may be because of a side effect of one of these variables. Take a look at all the <code>export</code> commands within <code>RunIPhoneUnitTest.sh</code> to see what's actually going on.<br>
<br>
<h2>Leaks</h2>
By default the iPhone unit tests will run leaks after all the tests have completed.  This can be turned off by setting the <code>GTM_DISABLE_LEAKS</code> environment variable before you execute the <code>RunIPhoneUnitTest.sh</code> script. Out of the box, NSZombies will also be enabled. This however interferes slightly with leaks and makes it difficult to get good backtraces and context. If you want the backtraces and context, set the <code>GTM_DISABLE_ZOMBIES</code> environment variable before you execute the <code>RunIPhoneUnitTest.sh</code> script. All leaks will appear as warnings on the build console.<br>
<br>
<h2>Termination</h2>
Some of Apple's tools (such as Instruments) don't want the app to terminate underneath them. By default the iPhone unit test app will terminate when it has finished it's run. Set the <code>GTM_DISABLE_TERMINATION</code> environment variable if you want to disable termination and just have the unit test "run" until you are done with it.<br>
<br>
<h2>Keychain Testing</h2>
If your code uses the keychain, you may need to set <code>GTM_DISABLE_IPHONE_LAUNCH_DAEMONS=0</code> before you run your unit tests. See RunIPhoneUnitTest.sh for details on this flag and why it should be set.