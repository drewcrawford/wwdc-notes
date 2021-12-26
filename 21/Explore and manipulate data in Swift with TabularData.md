# Introduction
What is tabular data?  Organized in rows and columns.

TabularData => new framework
Unstructured data.  
Exploration.  What information is there?  Values, types, etc.
Manipulation: transform the dataset into a form that is best-suited.  Dates as data instead of sttring.  etc.

Why?  
* Grouping data (by criteria)
* Joining data
* Splitting/segmenting
* Data pipelines (ML)

Table: DataFrame.  Every column can contain only a specific type.
Columns can hold any type, including custom types.
Index column.  Access rows by index.  
Filter: data unchanged
Sort: change data

Column.  Specific type.  Name.  
ColumnID.  All columns must have the same number of elements.  Missing elements as nil values.
Think of rows as proxy.  Doesn't have elements, just a reference.

## Building
* Init from literals
* Note that with literals, constrained to specific swift types.
* Every column must have same elements.

Or build 1 column at a time.

# Data exploration
* CSV
* JSON
Just call initailizer

## CSV options
* separator
* header row?
* Escaping
* Nils
* Partial loading (constructor argument)
* column selection and ordering
* Type inference
	* Tries numbers, booleans, and dates.  Can specify type per column.
* Date parsing
* ISO8601 parsed by default
* Otherwis euse a custom parsing strategy

## Writing data
print(dataFrame)
Not great for storing data.  `.writeToCSV`.  Uses default string conversion.  Generated CSV may not be something you can read back.

Writing options similar to reading.
`dataFrame[column: 1]
dataFrame[row: 1 ..< 4]
`
Note that due to filter, slices can be discontiguous.
Want to use `startIndex` and `endIndex` and `.index(after:)`.


## Demo: exploring the dataset
`.addDateParseStrategy()` to parse nonstandard date formats
Can't remove a column from a slice, instead promote to new DataFrame.

# Data transformation
* map
* transformColumn (in-place version)
* decode
* filled.  Replace all missing values with default value
* summary => quick overview of contents.  count, unique, mode, etc.  
* numeric summary: counts, mean, stdev, min, max, quartiles

## DataFrame transformations
* sort
* combineColumns => e.g. lat+lon = coordinate
* summary
* explode.  e.g. let's say some value in the row is an array `[1,2]`.  Split into separate rows

## Demo: augmenting a dataset
`grouped(by:)`.   Finds unique values, splits into corresonding groups.  

`.joined()`.  Combine two dataframes together using a key.  

# Best practices
* Load errors
	* Type assumptions are risky
	* Especially improtant for dates, as we might fall back to string
	* Failed to parse
	* Wrong number of columns
	* bad encoding
	* misplaced quotes

Performance.  
* Date parsing.  Consider delaying parsing.
* Grouping.  Use a basic swift type.
	* Consider combining columns if grouping by more than one column.
* Joining
	* Consider using basic swift type

## Demo

# Recap
* Start with exploration
* Transform and augment data to fit your needs
* ?

