# What are dynamic apps?
Personalized experiences.

You can do this with heuristics and predefined rules.  But maybe it's no good.

ML enables you to create models that learn directly from user data.  This may be more generalizable and suit more users.

What tools can you use?

# Create ML
Easy to create models by "simply" selecting training data and training.

Rich templates.

Provides accelerated training of ML models

Introduced in Mojave.

Bringing this framework to iOS 15 and iPadOS 15.

Available on device, your apps can do various things.

# Training with CreateML
* On-device model creation
* Learn from user's data
* Privacy preserved

# Examples
* Image classifier
* Text classifier
* Hand action classifier (new)

# Demo
## Training a style transfer model
1.  Single style image
2.  Set of content images
3.  Decide images vs video type
4.  Set style strength and style density
5.  Set number of iterations

[[Build Image and Video Style Transfer models in Create ML]]

```swift
// define training data source
let data = MLStyleTransfer.DataSource.images(styleImage: styleUrl, contentDirectory: contentUrl)

// define session parameter
let sessionParameters = MLTrainingSessionParameters(sessionDirectory: sessionUrl)

// define training job
let job = try MLStyleTransfer.train(trainingData: data, sessionParameters: sessionParameters)

// dispatch training job 
// save out model upon receiving successful completion, compile for later use
// make prediction with CoreML model
try model.write(to: writeToUrl)
let compiledURL = try MLModel.compileModel(at: writeToUrl)
let mlModel = try MLModel(contentsOf: compiledURL)
let inputImage = try MLDictionaryFeatureProvider(dictionary: ["image": image])
let stylizedImage = try mlModel.prediction(from: inputImage)
```

What if your app does not interact with such datatypes?

How you can make your app dynamic?

# Classifiers/regressors

Classifier: pick particular classes
Regressor: predicts numerical value

Boosted tree, decision tree, linear, random forest

Help service intelligent restaurant suggestions.  

3 kinds of information in a table.

1.  Content (keywords)
2.  Context (time of day)
3.  Interaction (past orders)

Personalized experience.

## Creating a model in code

1.  Prepare data
2.  Train
3.  Prediction

```swift
func featuresFromMealAndKeywords(meal: String, keywords: [String]) -> [String: Double] {

    // Capture interactions between content (the dish keywords) and context (meal) by
    // adding a copy of each keyword modified to include the meal.
    let featureNames = keywords + keywords.map { meal + ":" + $0 }
    
    // For each keyword, create an entry in a dictionary of features with a value of 1.0.
    return featureNames.reduce(into: [:]) { features, name in
        features[name] = 1.0
    }
}
```

```swift
var trainingKeywords: [[String: Double]] = []
var trainingTargets: [Double] = []

for item in userPurchasedItems {
    // Add in the positive example.
    trainingKeywords.append(
       featuresFromMealAndKeywords(meal: item.meal, keywords: item.keywords))
    trainingTargets.append(1.0)
            
    // Add in the negative example.
    let negativeKeywords = allKeywords.subtracting(item.keywords)
    trainingKeywords.append(
       featuresFromMealAndKeywords(meal: item.meal, keywords: Array(negativeKeywords)))
    trainingTargets.append(-1.0)
}
```

Negative exmaples allow the model to learn user preferences.

```swift
// Create the training data.
var trainingData = DataFrame()
trainingData.append(column: Column(name: "keywords" contents: trainingKeywords))
trainingData.append(column: Column(name: "target", contents: trainingTargets))

// Create the model.
let model = try MLLinearRegressor(trainingData: trainingData, targetColumn: "target")
```

Column I'm trying to predict is the target column.

```swift
// Setup the data to run an inference on.
var inputData = DataFrame()
inputData.append(column: Column(name: "keywords", contents: dishKeywords))

// Call predictions on the trained model with the data.
let predictions = try model.predictions(from: inputData)
```

# Best practices.

* General ML guidelines
* Training workflow (async, checkpointing)
* Integration with your apps (power, memory, storage concerns, etc)

[[Design Great ML experiences - 19]]
[[Control training in Create ML with Swift]]

# Wrap up
* Create ML framework now on iOS
* Make personal, customized apps
* Preserve user privacy
* Train and apply models in-app
* Easy to update with little code



