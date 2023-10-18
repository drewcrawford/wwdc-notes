Find out how Create ML can help you do even more with machine learning models. Learn about the latest updates to image understanding and text-based tasks with multilingual BERT embeddings. Discover how easy it is to train models that can understand the content of images using multi-label classification. We'll also share information about interactive model evaluation and the latest APIs for custom training data augmentations. To learn more about the latest updates to machine learning, watch “Explore Natural Language multilingual models” and “Improve Core ML integration with async prediction" from WWDC23.

Our goal is to give you the tools to build great apps without the overhead.

State of the art models that power many features, such as photos search,e tc.

Build on models included int eh OS.

# Create ML improvements

Text classification.  Recognize patterns in natural language text.

Choose the transfer learning algorithm that uses a pertained model as a feature extractor.  Billions of labeled text examples.

BERT.  New option in create ML app.  BERT is multilingual, so your training data can contain more than one language.  Boost the accuracy of your model text classifiers.

IOS 17, iPadOS 17, macOS Sonoma.
[[Explore natural language multilingual models]]

## Image classification
What is the best label to describe my image?

Pretrained model to extract relevant information from images.  ANSA.  Image understanding model sin the OS continue to evolve.  See our article.

New feature extractor option.  Smaller output embedding size.  Also boosts the accuracy of your classifier, reduce the memory needed, etc.
# Multi-label classification

Recall that single-label image classification predicts the best label.

Multi label image classifier can predict set of objects for your images.

Build a classifier that detects multiple plants in your scenes.  

Organize your annotations in a json file.  

Demo.

To start, I’ll drag in my training data.  

Option to drag in validation data.  Randomly split the training data for now.  Use the default number of iterations and leave out annotations.

In general, I want to maximize MAP (mean average precision) because it means higher precision and recall.  

Calculates metrics for each label.  Positives, negatives, precision, recall, etc.

Prediction is correct if the confidence is above the threshold.  

Code.
Create a vision model from cormel model.
Image request handler from source image
Perform request
Retrieve the classification observations using precision/recall value that I’m interested in.

[[Understanding images in Vision Framework - 19]]



# Data augmentation

Transformations like flip, crop,e tc.

* boosts model quality
* Improves generalization
* Slows training

[[Get to know Create ML Components]]
[[Compose advanced models with Create ML Components]]

This year, we’re adding new augmentation apis.

This may be familiar to you.

1.  Augmenter.  Result builders
2. Add transformations to transform your data.
3. Applies a transformation with a given probability
4. Horizontal flipper transformer, etc.

Important to carefully consider the nature of your data.  You won’t encounter an upside down succulent, so you don’t really need a vertical flip.

Or traffic signs, flipping may no longer correctly describe your image.

Randomly rotate my images.

Each transformation is applied in sequence.

I’ve only scratched the surface.  

What if you want to go even further?  Let’s go through an example of how you can build custom transformationsand use them to construct your images.

Example item that augments by applying a random background.

Input image over the randomly selected background.  Flipping, rotating, cropping happen before placing the succulent in different backgrounds.

Generate some interesting images, etc.

It makes more sense to train progressively, using the update method.


Shuffle before augmenting so that in each iteration the batches contain images.

A sync sequence -> transformations happen lazily.

New set of images on each iteration.  Depending on your augmenter.  Model can generalize instead of iterations.

Stop training when validation accuracy stops improving.  

# Wrap up
* create ML improvements
* Multi-label classification
* Build your own augmentations

# Resources
* https://developer.apple.com/documentation/coreml
* https://developer.apple.com/documentation/CreateML
* https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app
