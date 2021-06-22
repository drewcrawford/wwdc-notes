# Hosting documentation
[[Meet DocC documentation in Xcode]]

Can export from documentation window and share.  

To learn more, see talk.

## What is a documentation archive?
Container that holds all the data to read in xcode and host online.

Single page view.js webapp.
## Two types of documentation requests

* Requests for documentation and tutorial pages (`/documentation` or `/tutorials`)
 * Requests for files and data in the documentation archive

Example.  

* https://hostname.com/documentation/slothcreator
* https://hostname.com/tutorials/slothcreator

Serve the page.  lol.

Both the html and the content use request URLs that match the folder structure of the documentation object.

Already copied the doc archive.  Added an empty `.htaccess` file.

```
# Enable custom routing.
RewriteEngine On

# Route documentation and tutorial pages.
RewriteRule ^(documentation|tutorials)\/.*$ SlothCreator.doccarchive/index.html [L]

# Route files within the documentation archive.
RewriteRule ^(css|js|data|images|downloads|favicon\.ico|favicon\.svg|img|theme-settings\.json|videos)\/.*$ SlothCreator.doccarchive/$0 [L]
```

I have no idea why we are doing this instead of just moving files into a logical location.

# Automating builds

How to automate building/updating doc archive?  

Can run my script when changes are made.  
Can now build doc on the comand line.  `xcodebuild docbuild`.

When you build documentation, it works like a standard build that also builds documentation.  During the build, swift compiler gathers information which is passed to `DocC`.  

Also comes from DocC catalog, see related talk

[[Elevate your DocC documentation in Xcode]]

If your target has dependencies to other framworks, etc., we build all dependencies.

```bash
# Build documentation for the project.
xcodebuild docbuild                    \
  -scheme "SlothCreator"               \
  -derivedDataPath MyDerivedDataFolder
  
# Find all the built documentation archives
# to copy them to another location.
find MyDerivedDataFolder               \
  -name "*.doccarchive"
```

Like the build action, pass the scheme to build.

Not required, but you can specify `derivedDataPath` for products and documentation.

```bash
#!/bin/sh

# Build the SlothCreator documentation.
xcodebuild docbuild                  \
  -scheme "SlothCreator"             \
  -derivedDataPath MyDerivedDataPath
  
# Copy the documentation archive to ~/www where we
# host the SlothCreator website and documentation.
find MyDerivedDataPath               \
  -name "*.doccarchive"              \
  -exec cp -R {} ~/www \;
```

# Wrap up
* Share or host documentation archives
* Build documentation for automation

[[Meet DocC documentation in Xcode]]
[[Elevate your DocC documentation in Xcode]]
[[Build interactive tutorials using DocC]]

