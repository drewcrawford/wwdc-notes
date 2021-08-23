In 2019, we made it possible to train sound classification models with createML.

We leverage this to do sound recognition for accessiblity features.

New this year, we have a soundclassifier built-in.

Supported on all platforms

# Developing a sound classifier
* annotated data not needed
* Specialized expertise
* compute power

It only takes a few lines of code to enable sound classification.

Over 300 sound classes.
(categories)
Animals,
music
human sounds

# Demo
Find sound inside a file.

1.  Acquire list of recognized sounds
2.  Prompt user for sound to search
3.  For each selected file in finder...
	1.  Find sound in file
	2.  Extract and show clip

[[Meet shortcuts for macOS]]

```swift
func getListOfRecognizedSounds() throws -> [String] {
    let request = try SNClassifySoundRequest(classifierIdentifier: .version1)
    return request.knownClassifications
}
```

## Perform sound classification
1.  `SNClassifySoundRequest`
2.  `SNAudioFileAnalyzer`
3.  Custom observer


```swift
let request = try SNClassifySoundRequest(classifierIdentifier: .version1)

let analyzer = try SNAudioFileAnalyzer(url: url)

var observer: SNResultsObserving // TODO

try analyzer.add(request, withObserver: observer)
analyzer.analyze()
```
observer
```swift
class FirstDetectionObserver: NSObject, SNResultsObserving {
    var firstDetectionTime = CMTime.invalid
    var label: String
    
    init(label: String) {
        self.label = label
    }
    
    func request(_ request: SNRequest, didProduce result: SNResult) {
        if let result = result as? SNClassificationResult,
           let classification = result.classification(forIdentifier: label),
           classification.confidence > 0.5,
           firstDetectionTime == CMTime.invalid {
                firstDetectionTime = result.timeRange.start
        }
    }
}
```

## Time of detection
When you classify audio, the signal is broken up into overlapping windows.  For each window, you'll get a result that tells you what sounds were detected and how confidently.  Will also get a timerange to tell you what part of the audio was classified.

Timerange can be impacted when duration of a window changes.  Can customize window to be big or small based on usecase.

Short windows work well with short sounds.  Because you can capture all the features within a small window of time.  Small window doesn't cut out any important information.  The advantage of using a small window duration is that it allows you to closely pinpoint the moment at which a sound occurred.

But may not be appropriate when working in longer sound.  e.g. siren contains rising and falling pitches.  All pitches together can help sound classification correctly detect the sound.  In general, use the window duration long enough to capture all parts of the sound oyu're interested in.

`request.windowDuration`.  Note that not all window durations are supported.  Different classifiers might support different durations.  Check `windowDurationConstraint` property.  Built-in classifier supports half-second to 15 seconds.  Duration of >=1sec is a great starting point.



## Detection thresholds
How to choose thresholds?

* Classifier can detect multiple sounds at the same time.  When this happens, you may notice that several score with high confidence.

Unlike with a custom model with coreml, label scores do not add to 1.0.  Independent, don't compare against one another.  Because they're independent may find it useful to choose different thresholds for each sound.  

When you select a threshold, find a value taht achieves the right balance of false positives and false negatives.  confidence scores may change when you change window duration.

## Acoustically similar sounds
* water faucet vs bathtub filling
* percussion vs fireworks
* lawn mower vs car engine

Consider being selective with what sounds you're looking for.

# CreateML
Our model contains a lot of knowledge.  This can be used in your custom models.

1.  Feature extractor
2.  Classifier model

Feature extractor ("embedded model") is the backbone network.  Transforms audio into low-dimensional space.

Organize acoustically similar sounds into nearby locations in the space.  

Classifier model takes the output of the feature extractor and computes class probabilities.

We're makign the builtin feature extractor available to you => "Audio feature print".  When you create your own custom model, it will be paired.  Your model benefits from knowledge in the classifier.

Advantages
* Improved accuracy
* Smaller model size
* Faster inference
* Faster training
* Flexible input dimension

New default feature extractor.  Window duration defaults to 3s, can choose 0.5s-15s
[[Training Sound Classification Models in CreateML - 19]]

# Wrap up
* Built-in sound classifier
* Audio Feature Print

https://developer.apple.com/documentation/soundanalysis/classifying_live_audio_input_with_a_built-in_sound_classifier
https://developer.apple.com/documentation/soundanalysis
