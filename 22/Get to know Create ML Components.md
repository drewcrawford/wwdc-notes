#createml  simple API for training ML models.

At wwdc 2021, we presented two great talks

[[Classify hand poses and actions with Create ML]]
[[Build dynamic iOS apps with the Create ML framework]]

But how to customize a task beyond CreateML?  How abuot building a different task?  Using ocmponents, you can compose tasks in new and creative ways.

# Breaking up ML tasks
## Image classifier
Labeled images to trian a model.  Images of cats and dogs with respective laberls.  How are images transformed at each step?  Let's expand the `MLImageClassifier` task.

Conceptually,
* feature extractor
* classifier

CreateML components gives you access to these components independently.  Add/remove/switch components.  Represent these as boxes.  Arrows represent the flow of data.  Let's see the feature extraction.

Generally, featrure extractors reduce dimensionality of the input, keeping only the interesting parts.  Looks for patterns in an image.  We use Vision FeaturePrint, provided by Vision framework.

Classifier.  Set of exapmles to learn a classification.  e.g. logistic regression, boosted trees, neural networks.

## Why?
We want to expand possibilities.  maybe you want to do some preprocessing by increasing the contrast.  Or normalize all images so they have uniform brightness.  Maybe you want to try a different feature extractor.  Or a different classifier.

Support for ML components in fall releases.  macOS, iOS, iPadOS, tvOS.  Compose new models etc.

Leverage across our platforms.

## Transformers and estimators.
* transformer => type that is able to perform some transformation.  Defines an input and otuput type.  e.g. image-feature extractor.
* estimator => learns from data.  Taskes input examples, does processing, and produces a transformer.  We call this "fitting".

# Composition
How to build an image classifier from individual components.

Combine them using `.appending`.  This is where copmonents provide unlimited possibilities.  Use a fully ocnnected neural network as a classifier instead of regression.  Or use a custom feature extractor in a core ML model.  e.g., ResNet50.

When composing two comonents, output of the first must match input of the second.  Ensure the types match.

When your composed estimator has both transformers and estimators, lik in the case of a classifier, only the estimators are fitted.  The transformers  are important though.

```swift
let data: [AnnotatedFeature<CIImage, String>] = ...
let model = try await task.fitted(to:)
```

do predictions, etc.  Can load parameters from disk.  API is same in both cases.  Since I am training a classifier, result is a classification distribution.  Includes ap robability for each label.  Here I'm just printing the most likely label for an image.

The `fitted` method also provides a mechanism to observe training methods.  Passing validation data and printing the validation accuracy.  Only supervised estimators provide validation metrics.  Once you train a model, save the learned parmaeters either to reuse or to deploy.

That's composition.
# Custom image tasks
Let's write a new kind of task.  What if... you want to train amodel to score images?  Let's give fruit a score based on how ripe it is.

This requires regression, not classification.  So we give a score to images.  Each image has a value between 1 and 10.  

A regressor is similar to a classifier.  But our estimator will be a regressor, not a classifier.  This will be easy.

```swift
import CoreImage
import CreateMLComponents

struct ImageRegressor {
    static let trainingDataURL = URL(fileURLWithPath: "~/Desktop/bananas")
    static let parametersURL = URL(fileURLWithPath: "~/Desktop/parameters")

    static func train() async throws -> some Transformer<CIImage, Float> {
        let estimator = ImageFeaturePrint()
            .appending(LinearRegressor())

        // File name example: banana-5.jpg
        let data = try AnnotatedFiles(labeledByNamesAt: trainingDataURL, separator: "-", index: 1, type: .image)
            .mapFeatures(ImageReader.read)
            .mapAnnotations({ Float($0)! })

        let (training, validation) = data.randomSplit(by: 0.8)
        let transformer = try await estimator.fitted(to: training, validateOn: validation)
        try estimator.write(transformer, to: parametersURL)

        return transformer
    }
}
```

Let's demo this with some code.  How to write a custom image regressor.  Define a struct to capture the code.

mapAnnotations to go from strings to floating point values.  80% for training, the rest for validation.  

Some things I can improve.  For starters, I am passing the validdation dataset, but not observing the error.  So let's do that.

```swift
import CoreImage
import CreateMLComponents

struct ImageRegressor {
    static let trainingDataURL = URL(fileURLWithPath: "~/Desktop/bananas")
    static let parametersURL = URL(fileURLWithPath: "~/Desktop/parameters")

    static func train() async throws -> some Transformer<CIImage, Float> {
        let estimator = SaliencyCropper()
            .appending(ImageFeaturePrint())
            .appending(LinearRegressor())

        // File name example: banana-5.jpg
        let data = try AnnotatedFiles(labeledByNamesAt: trainingDataURL, separator: "-", index: 1, type: .image)
            .mapFeatures(ImageReader.read)
            .mapAnnotations({ Float($0)! })
            .flatMap(augment)

        let (training, validation) = data.randomSplit(by: 0.8)
        let transformer = try await estimator.fitted(to: training, validateOn: validation) { event in
            guard let trainingMaxError = event.metrics[.trainingMaximumError] else {
                return
            }
            guard let validationMaxError = event.metrics[.validationMaximumError] else {
                return
            }
            print("Training max error: \(trainingMaxError), Validation max error: \(validationMaxError)")
        }

        let validationError = try await meanAbsoluteError(
            transformer.applied(to: validation.map(\.feature)),
            validation.map(\.annotation)
        )
        print("Mean absolute error: \(validationError)")

        try estimator.write(transformer, to: parametersURL)

        return transformer
    }

    static func augment(_ original: AnnotatedFeature<CIImage, Float>) -> [AnnotatedFeature<CIImage, Float>] {
        let angle = CGFloat.random(in: -.pi ... .pi)
        let rotated = original.feature.transformed(by: .init(rotationAngle: angle))

        let scale = CGFloat.random(in: 0.8 ... 1.2)
        let scaled = original.feature.transformed(by: .init(scaleX: scale, y: scale))

        return [
            original,
            AnnotatedFeature(feature: rotated, annotation: original.annotation),
            AnnotatedFeature(feature: scaled, annotation: original.annotation),
        ]
    }
}
```

Will use mean absolute error for validation.  

Error was high.  I need more images.  But let's try augmenting the dataset.  To do this, we use the method shown above.  FlatMat flattens the array of augmentations to a single array.

Only apply when fitting, not when doing  prediction.


I want to use the vision framework to crop to the salient object.  Model may get confused by background objects.  Using vision framework, I can crop the image.

[[Understanding images in Vision Framework - 19]]

Implement the `applied` method.  Now that I have a custom `Transformer`, add to my iamge request.

Just need to use my custom transformer before feature extraction.  Now that saliencey is part of my task definition, it will be used to crop training images and inference.  Share impelmentation between training and inference.
recap
* creating composed tasks is easy
* Use `AnnotatedFiles` when loading annotated files
* Use `mapFeatures` and `mapAnnotations` as necessary
* Use `randomSplit` and provide a validation dataset
* write your trained parameters


# Custom tabular tasks
Another type of task.  Tabular tasks.  These are tasks that use tabular data.  Characterized by having multiple features of different types.  Numerical and categorical data.  A popular example is house-pricing data.  Area, age, neighborhood, type of building, etc.  Learn to predict a value: the sales price.

In 2021 we introduced TabularData framework.  use the framework together with CreateML components to build and train tabular classifiers and regressers.

[[Explore and manipulate data in Swift with TabularData]]

Per-column operations: `ColumnSelector`.  distribution, rang eof values, etc.  CreateML components lets you do this using `ColumnSelector`.

Some regressors benefit from ahving a better representation.  ex, volume is close to anormal distribution.  This is a great example of a dataset that can benefit from normalization.  Pass column names I want to normalize to a ColumnSelector and then use a standard scalar.

> My values now have a mean of 0 and a stdev of 1.

## One-hot encoding
Encoding categories of data using an array to indicate the presence of each category.

bronze => 1,0,0
silver => 0,1,0
gold => 0,0,1

Use this method when there are only a few categories.  
## ordinal encoding
Gives a consecutive number for each category.

Use this if there are more categories.

## building
```swift
import CreateMLComponents
import Foundation
import TabularData

struct TabularRegressor {
    static let dataURL = URL(fileURLWithPath: "~/Downloads/avocado.csv")
    static let parametersURL = URL(fileURLWithPath: "~/Downloads/parameters.pkg")

    static let priceColumnID = ColumnID("price", Double.self)

    static var task: some SupervisedTabularEstimator {
        let numeric = ColumnSelector(
            columns: ["volume"],
            estimator: OptionalUnwrapper()
                .appending(StandardScaler<Double>())
        )
        let regression = BoostedTreeRegressor<String>(
            annotationColumnName: priceColumnID.name,
            featureColumnNames: ["type", "region", "volume"]
        )

        return numeric.appending(regression)
    }

    static func train() async throws -> some TabularTransformer {
        let dataFrame = try DataFrame(contentsOfCSVFile: dataURL)
        let (training, validation) = dataFrame.randomSplit(by: 0.8)
        let transformer = try await task.fitted(to: DataFrame(training), validateOn: DataFrame(validation)) { event in
            guard let validationError = event.metrics[.validationError] as? Double else {
                return
            }
            print("Validation error: \(validationError)")
        }
        try task.write(transformer, to: parametersURL)
        return transformer
    }

    static func predict(
        type: String,
        region: String,
        volume: Double
    ) async throws -> Double {
        let model = try task.read(from: parametersURL)
        let dataFrame: DataFrame = [
            "type": [type],
            "region": [region],
            "volume": [volume]
        ]
        let result = try await model(dataFrame)
        return result[priceColumnID][0]!
    }
}
```

Use a **boosted tree regressor** to predict the price.  ]

## Considerations
* columnSelector is a useful operation for tabular tasks
* Tree estimators are all tabular
* Could also used `AnnotatedFeatureProvider` to adopt non-tabular estimators.
	* see docs
* When doing predecitions, build a dataframe.
# Deployment
Model is code.  You need the task definition, even when loading trained parameters from a file.  Useful in some situations, but maybe you want to use coreml for deployment.  In this case, you leave the code behind.

* model is an asset
* optimized tensor operations

Some considerations though.
* Not all operations are supported in coreml
	* custom transformers, estimators, are not supported
	* only supports a few types like images and shaped arrays
* Convert to CoreML types
	* `try transformer.export(to:)`

To deploy your task definition instead, you should consider bundling them into a swift package

[[Swift packages Resources and Localization - 20]]

# Wrap up
* create new tasks using composition
* the sky is the limit
[[Compose advanced models with Create ML Components]]









```swift

```


* https://developer.apple.com/documentation/CreateML