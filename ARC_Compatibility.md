# ARC Compatibility #

When the library source files are compiled directly into a project that has ARC enabled, then ARC must be disabled specifically for the library source files.

To disable ARC for source files in Xcode 4, select the project and the target in Xcode. Under the target "Build Phases" tab, expand the Compile Sources build phase, select the library source files, then press Enter to open an edit field, and type  `-fno-objc-arc`  as the compiler flag for those files.

Apple's Greg Parker has [recommended](http://lists.apple.com/archives/objc-language/2011/Aug/msg00036.html) this approach as it ends up cleaner then trying to stick `ifdef`s through the code.