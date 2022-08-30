#createml 

Build your own ML tasks.

Introduced in [[Get to know Create ML Components]].  Transformers, estimators.  In this session, I want to go way beyond teh basics and demosntrate what's possible.

# Temporal components
at 2020, we introduced action classification in create ml.  Allows you to classify actions from videos.  [[Build an action classifier with create ml]]

## Data stream
Swift's AsyncSequence makes this easy.  See [[Meet AsyncSequence]]

With Create ML ocmponents, you can read your video as an async sequence of frames.  provides a way of iterating over the frames as they are received from the video.  For example, I can easily transform each video frame asynchronously, using the map method.  Useufl when you want to process frames one at a time.  Process multiple frames at a time?  That's where temporal transfoerms come in. 

How to downsample frames to speed up a video.  Take an Async Sequence and return a downsampled sequence.  Group frames into windows, which is important for ocunting repeitions.  Use a sliding window transformer.  Specify window length, and a stride, which is how yuo control the sliding window.  Input is an async sequence, andoutput is a windowed sequence.

A temporal transformer provides a way to transform one asyncsequence into another.
# Action reptition counting
Compose transformers and temporal transformers together.  Extract poses using `HumanBodyPoseExtractor`.  Output is an array of human body poses.  Vision framework will extract poses.

Images contain multipel people.  Common for group workouts.  Output is an array of poses.  I'm only interested in counting action repetitions.  So I'll compose the human body pose extractor, with a pose selector.

Pose selector takes an array of poses, and a selection strategy, and returns a single pose.  For this example, I'll use `rightMostJOintLocation`.  

Group poses into windows.  Sliding window transformer.  Use a window legnth and stride of 90, to generate nonoverlapping windows of 90 poses.  Recall that a sliding window transformer is temporal.  Expected input is now an async sequence of frames.  Finally, I'll append a human body action counter.  This temporal transformer consumes a windowed async sequence of poses, and returns a cumulative count.  By now, count is a lfoating point number.  Task counts partial actions too.

Even better to count reptitions live in an app.  Track c urrent workouts.  

1.  Use `readCamera` method.  Async sequence of frames
2. Adjust stride parameter to `15` frames.  
# Incremental fitting
Training custom models that rely on temporal data.

[[Training Sound Classification Models in CreateML - 19]]
[[Discover built-in sound classification in SoundAnalysis]]


MLSoundClassifier is still the easiest wya to train a custom sound classifier model.  When you need moer customizability and control, use thw components.

1.  AudioFeaturePrint (feature extractor)
2. classifier (of your choice)

AudioFEaturePrint is a temporal transformer, that extracts features from a sequence of buffers.  Similar to a sliding window transformer, it windows the async sequence then extracts features.  A few classifiers to choose from, here i'll choose a logistic regression classifier.

Compose together with feature extractor.  Fit the classifier.  For more info, see [[Get to know Create ML Components]]

Building models can be iterative.

1.  Data
2. training
3. evaluation
4. goto 1

Retrianing your model from scratch is time-consuming.  Here's how to save time when training your models with newly discovered data.

Preprocess your training data separately from fitting your model.  We extract audio features separately from classifier fitting.  Whenever you have a series of transformers, followed bby an estimator, preprocess the input.

Call `task.preprocess`.  Then fit the model on the preprocessed features.  

I have the flexibility to only extract audio features for the new data.  Just the first example of where preprocessing can save time.

Let's change classifier parameters without doing feature extraction.  if I've alreayd extracted featuers, I'll modify `l2Penalty=0.1`.  Then append new classifier to old feature extractor. Important not to change feature extractor when tuning your estimator, that would invalidate the previously extracted features.

Your app may have limited memory constraints.  You can use createML components to train a model by loading batches into memory at a time.  Replace classifier with updatable classifier.  Ex, fully connected NN classifier, which I can easily use instead of the logistic regression classifier, which is not updatable.

Now I"l write a training loop.

1.  Create a default initialized model.  
2. Extract audio features before the training starts.  I don't want to extract featuers each iteration.
3. for iteration in 0..<10
4. import Algorithms
	5. [[Meet the Swift Algorithms and Collections packages]]
6. Chunk size is the number of featuers that are loaded into memory at once.
7. Iterate over batches and call `update`.

Let's implement early stopping.

Let's talk about model checkpointing.  Can save progress rather than waiting until the end.  Can even use checkpointing to resume training, when your model takes a long time to train.  Write out yuor model in the training loop.  Every few iterations.

# Wrap up
* temporal components
* Action repetition counting
* Incremental fitting



https://developer.apple.com/documentation/createmlcomponents/counting_human_body_action_repetitions_in_a_live_video_feed
https://developer.apple.com/documentation/CreateML