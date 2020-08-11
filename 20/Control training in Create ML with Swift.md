#createml 

# App vs framework
Under the hood, the app is powered by createML framework.  For macOS Big Sur, we're bringing apis etc.

Customize the way you create ML models and leverage the same tech that powers the app.

# Training process
Collect data.  Look at training options available to you.  Using CreateML, ... default settings...

Preprocess.  Raw audio transformed by taking spectogram, or video extract points.

Training.  Cover later.

Evaluate.  Present with new data.  Use a different set of data that the model hasn't seen during traning.  Will help you determine if your mode is generalizing.

Model development is an iterative process so you can return to setup or training.  

Deploy.  Once you are satisfied, you can use your model to perform predictions.

# Control
Mainly having the ability to pause/resume.  But also observe/alter the training process while it's happening.

Training models take time.  Without training control, you ahve to start form scratch and choose a larger number of iterations.

Results are specific to what you're trying to achieve.  Using a built-in mechanism will not necessarily give the best results to your specific usecase.

[[Build Image and Video Style Transfer models in Create ML]]

It's up to you to decide when this is good enough.

Choosing when to stop is specific to your needs.  Prototype vs production.

Can define custom stopping criteria, so not limited to the number of iterations.  Target accuracy, specific amount of cpu time for training, etc.

```swift
let model = try MLActivityClassifier(...)
let job = try MLActivityClassifier.train(..., sessionParameters: sessionParameters)
// Session parameters can be provided to `train` method.
let sessionParameters = MLTrainingSessionParameters(
    sessionDirectory: sessionDirectory, //created if it doesn't already exist.  resume previous session with this data.
    reportInterval: 10, //smaller intervals give more frequent updates but may come at a performance cost
    checkpointInterval: 100,
    iterations: 1000 //training will stop here.  But can always increase this number in later calls.
)

let job = try MLActivityClassifier.train(..., sessionParameters: sessionParameters)
```

# Job
`MLJob`.  Various publishers , progress, checkpoint, and result publishers

[[Introducing Combine - 19]]
[[Combine in Practice - 19]]

Also provides a cancel method so you can stop training easily at any point.

```swift
// Register a sink to receive the resulting model.
job.result.sink { result in
    // Handle errors (or success)
}
receiveValue: { model in
    // Use model
}
.store(in: &subscriptions)
```

```swift
// Observing progress details
job.progress.publisher(for: \.fractionCompleted)
    .sink { [weak job] fractionCompleted in
        guard let job = job, let progress = MLProgress(progress: job.progress) else {
            return
        }
        print("Progress: \(fractionCompleted)")
        print("Iteration: \(progress.itemCount) of \(progress.totalItemCount ?? 0)")
        print("Accuracy: \(progress.metrics[.accuracy] ?? 0.0)")
    }
    .store(in: &subscriptions)
```

# Demo
```swift
let style = NSImage(byReferencing: styleImageURL)
let validation = NSImage(byReferencing: validationImageURL)

var iterations = 500
var progressInterval = 5
var checkpointInterval = 5
let sessionDirectory = URL(fileURLWithPath: "\(NSHomeDirectory())/\(experimentID)")

let sessionParameters = MLTrainingSessionParameters(sessionDirectory: sessionDirectory,
                                                    reportInterval: progressInterval,
                                                    checkpointInterval: checkpointInterval,
                                                    iterations: iterations)

let trainingParameters = MLStyleTransfer.ModelParameters(
  	algorithm: .cnn,
    validation: .content(validationImageURL),
    maxIterations: iterations,
    textelDensity: 416,
    styleStrength: 5)
```

```swift
var subscriptions = [AnyCancellable]()

let job = try MLStyleTransfer.train(trainingData: dataSource,
                                    parameters: trainingParameters,
                                    sessionParameters: sessionParameters)

job.result.sink { result in
    print(result)
}
receiveValue: { model in
    try? model.write(to: sessionDirectory)
}
.store(in: &subscriptions)
```


```swift
//progress
job.progress
    .publisher(for: \.fractionCompleted)
    .sink { completed in
        
        _ = completed
        
        guard let progress = MLProgress(progress: job.progress) else { return }
        
        if let styleLoss = progress.metrics[.styleLoss] { _ = styleLoss }
        
        if let contentLoss = progress.metrics[.contentLoss] { _ = contentLoss }
        
    }
    .store(in: &subscriptions)
```

```swift
//cancel and resume
job.cancel()

//just use the same directory we used before
let resumedJob = try MLStyleTransfer.train(
    trainingData: dataSource,
    parameters: trainingParameters,
    sessionParameters: sessionParameters)

resumedJob.progress
    .publisher(for: \.fractionCompleted)
    .sink { completed in
        _ = completed
        
        guard let progress = MLProgress(progress: resumedJob.progress) else { return }
        if let styleLoss = progress.metrics[.styleLoss] { _ = styleLoss }
        if let contentLoss = progress.metrics[.contentLoss] { _ = contentLoss }
    }
    .store(in: &subscriptions)

resumedJob.result.sink { result in
    print(result)
}
receiveValue: { model in
    try? model.write(to: sessionDirectory)
}
.store(in: &subscriptions)
```


# Checkpoints
Capture the stat eof your model over time.  See how training compares over time.

`MLCheckpoint` Training or feature extraction checkpoint
* generated automatically when training asynchronously
* Easy to resume (useful for playgrounds)
* Generated during Preprocess and Train phases.

## preprocessing
Take individual files or rows in a table and process each one.  Progress is measured by how many elements are processed.

Every few elements, we store a checkpoint.

## training
Also iterative.  Model improves in discrete steps called iterations.  Running a batch of data, computing loss and updating weights.

Checkpoitns is the state of a model in a particular iteration.  Training preserves all checkpoints you create unlike preprocessing.

Metrics like loss/accuracy are stored along with checkpoints.

```swift
//observing checkpoints
let job = try MLActivityClassifier.train(..., sessionParameters: sessionParameters)

// Register for receiving checkpoints.
job.checkpoints.sink { checkpoint in
    // Process checkpoint
}
.store(in: &subscriptions)
```

```swift
// Generate a model from a checkpoint
guard checkpoint.phase == .training else {
    // Not a training checkpoint, can't create model yet.
    return
}

let model = try MLActivityClassifier(checkpoint: checkpoint)
try model.write(to: url)
```

## availability
|                         | Feature extraction | training |
|-------------------------|--------------------|----------|
| Image classification    | yes                |          |
| Sound classsification   | yes                |          |
| Action classification   | yes                | yes      |
| Object detection        |                    | yes      |
| style transfer          |                    | yes      |
| Activity classification |                    | yes      |

## Sessions

Aggregate of all checkpoints and their metadata.  `MLTrainingSession`

* List of checkpoints
* creation date
* training phase
* iteration number

```swift
//working with a session
let session = MLObjectDetector.restoreTrainingSession(sessionParameters: sessionParameters)

let losses = session.checkpoints.compactMap { $0.metrics[.loss] as? Double }
```

```swift
//remove checkpoints from a session
let session = MLObjectDetector.restoreTrainingSession(sessionParameters: sessionParameters)

// Save space by removing some checkpoints
session.removeCheckpoints { $0.iteration < 500 }
```

## demo

```swift
//visualizing style transfer checkpoints
job.checkpoints
    .compactMap { $0.metrics[.stylizedImageURL] as? URL }
    .map { NSImage(byReferencing: $0) }
    .sink { image in
        let _ = image
    }
    .store(in: &subscriptions)
```

```swift
//visualizing checkpoints with swiftUI + live view
job.checkpoints
    .compactMap { $0.metrics[.stylizedImageURL] as? URL }
    .receive(on: DispatchQueue.main)
    .map { NSImage(byReferencing: $0) }
    .sink { image in
        let _ = image
        
        let view = VStack {
            Image(nsImage: image)
                .resizable()
                .aspectRatio(contentMode: .fit)
            Image(nsImage: style)
                .resizable()
                .aspectRatio(contentMode: .fit)
            Image(nsImage: validation)
                .resizable()
                .aspectRatio(contentMode: .fit)
        }.frame(maxHeight: 1400)
        
        PlaygroundSupport.PlaygroundPage.current.setLiveView(view)  
    }
    .store(in: &subscriptions)
```

# Recap
* asynchronous training (new)
* checkpointing
* Combine publishers


