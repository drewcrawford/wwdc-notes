#clang

# Discovering bugs early
* Find bugs by scanning your code
* Catches bugs before testing and QA
* Works with Swift/ObjC projects (but C,C++,ObjC code only)

# New checks in Xcode 13
* Infinite loops
* Unused and redundant code
* Side effects in asserts
* C++ move misuses


# Customizing analysis
"Analyze during build" build setting.  Runs only on files that are modified.  Makes fast and easy.

Shallow vs deep.  Shallow limits scope to a single function.

Fine-tune checks that suit your project in build settings.  

Analyze single file with product=>perform action.  especially for modifying header files.

# Wrap up
* The analyzer is a powerful tool for finding bugs
* Try our improvements in Xcode 13

[[What's new in Clang and LLVM - 19]]
[[What's new in LLVM - 18]]

