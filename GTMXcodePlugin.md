


# Introduction #
There are two different GTM Xcode plugins. Please make sure to choose the right version.
They are both available [downloads](http://code.google.com/p/google-toolbox-for-mac/downloads/list).

## Installation ##
Both plugins should be installed in:

"~/Library/Application Support/Developer/Shared/Xcode/Plug-ins"

Note: the path might not exist, you'll need to create it in that case.

After installation, restart Xcode, and you should have a "About GTM Xcode Plugin" menu item under the Xcode menu. If you don't have this item, please check the location you installed it and try again. You may also want to check the console to see if anything interesting was spit out there.

## Bugs ##
If you run into any problems, please log them at:

http://code.google.com/p/google-toolbox-for-mac/issues/list

# Xcode 4 Version #
The GTM Xcode 4 plugin currently only adds a "Clean Up Whitespace" menu item to the end of the "Edit" menu to remove unnecessary end of line white space from text files. Hopefully we will add more features soon. It has only been tested against Xcode 4.2.

## Version History ##
  * 4.0.0 - Initial public release.

# Xcode 3 Version #
The GTM Xcode 3 plugin adds a couple of nice features to Xcode.
  1. A preference to clean up end of line white space from text files.
  1. A "Create Unit Test Executable" menu item to make it easier to debug unit tests.
  1. Some quick links under the "Help" menu to some useful documentation.
  1. Some menu items for working with [Coverstory](http://code.google.com/p/coverstory/)

## Version History ##
  * 10.0.1 - Initial public release.
  * 10.0.2 - Fixed some problems with custom absolute builds paths.
  * 10.0.3 - Now does a better job with determining OBJC\_DISABLE\_GC and working with hosts with relative paths for Custom Unit Test Executables.
  * 10.0.4 - Now picks up the active architecture properly when finding code coverage files.
  * 10.0.5 - Fix up small version numbering problem. No new features.
  * 10.0.6 - Now works on non-English systems (although with English UI. Feel free to submit localizations.) Works around NSBundle bug by passing full paths to the unit test bundles for Custom Unit Test Executables.