#swiftpm 

Developers would like to see more information, such as licenses.  

Standardize way of accessing metadata.  Package collections.

Course material, etc.  Enterprises can narrow down decisions and focusing on a trusted set of packages.

# Demo
Fixit to "search package collections".  

Release notes, etc.  

Default collection.  Algorithms, argument parser, etc.  Published by apple.

Makes use of these projects more seamless.  Default collection is updated regularly.
# Using collections
Importing packages is effortless.  What is a collection?

Json file.  Fetched via https.

* List of package URLs
* metadata.  Summary, version, etc

Fields:
* url
* readme
* summary
* versions

## Caching
Database for caching collections.  Access your configured collections from any tool, including CLI
swift.org/package-manager

## libSwiftPM
Including collection support.  Powers xcode as well.


# Creating a collection
Alamofire (?!?)
swift-format

* Name
* overview
* keywords
* author
* packages
* summary
* keywords
* versions
* excludedProducts
* readme URL

`package-collection-generate input.json collection.json --verbose --auth-token `

`package-collection-sign collection.json collection-signed.json developer-key.pem developer-cer.cer`

`swift package-collection add [url]`

`swift package-colleciton describe [url]`

`swift package-collection describe [package url]`

# Demo
# Wrap up
* Help with discovery of Packages
* Allow sharing curated lists of Packages
* Streamline the use of Packages

[[Adopting Swift packages in Xcode - 19]]
[[Creating Swift packages - 19]]

* https://github.com/apple/swift-package-collection-generator



