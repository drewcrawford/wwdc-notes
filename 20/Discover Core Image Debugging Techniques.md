* What `CI_PRINT_TREE` is
* How to enable and control it
* how to get the files
* how to interpret the files

This powers quicklook.

# How to enable
Set an environment variable from xcode
can also enable with `CI_PRINT_TREE="7 pdf" myexecutable`

# how to control
`CI_PRINT_TREE = "<graph type> <output type> <options>"`

## graph type
3 stages of a CI render

| 1 | Initial graph      | Useful for seeing what color spaces are used                                                  |
|---|--------------------|-----------------------------------------------------------------------------------------------|
| 2 | Optimized graph    | Useful for seeing how CI optimizes                                                            |
| 4 | concatenated graph | Useful for seeing how much memory is needed<br>(related to intermediate buffers CI allocates) |
| 7 | print graphs 1,2,4 | Useful for verbose logging                                                                    |

## output type
pdf or png.  
Will save the trees as documents to
* temporarry items directory on macos
* documentary directory on iOS, or falling back to temporary directory

If output type is not set, tree will print text format to `stdout`
Text can go to Console.app by setting `CI_LOG_FILE="oslog"`

## additional options
| `context==name`      | Only print for context with that name          |
|----------------------|------------------------------------------------|
| `frame-n`            | Print the nth render of each context           |
| `dump-inputs`        | Include the input images in the graph          |
| `dump-intermediates` | Include the intermediate in the graph (4 only) |
| `dump-outputs`       | Include the output of the graph (4 only)       |

# how to get files
On macOS, go to temporary directory.  Note that sandboxed apps will have a unique temporary directory.

On iOS, go to app's custom target properties.  Make sure "Application supports iTunes file sharing" key to YES.  Then you can find it in finder inside the app documents.

# how to interpret
* Inputs are at the bottom, output is at top
* green is for warp kernels, red is for color kernels
* colorspace names can be seen in the Initial tree
* ROIs show the region of each node that is needed for this render
* Use `4` and `dump-intermediates` to see intermediates
	* can help determine where a bug occurs
	* no intermediate implies caching was used
* Shows execution time of each pass
* pixel, pixel format.  

