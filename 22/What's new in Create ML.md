#createml 

Train MLModels.
App bundled with xcode, letting you quickly build and train models.

Available as as framework via SDK.
Easily automate model creation dynamically from your app.

[[Introducing the Create ML App - 19]]
[[Build dynamic iOS apps with the Create ML framework]]

# What's new, app
* Model evaluation
* Expanded capabilities
* Customization

1.  Idea
2. Data
Collecting and labeling images.

Evaluate how well it performs.  Here, think how accurate the model is on images which were not part of the training set.  Dependingo n evaluation, you may iterate on the model.

Focus on evaluation step.
When performing an evaluation, we often look at metrics testing new data, not in training.
* Accuracy
* strengths
* weaknesses

Use dropdown menus to add false positives, false negatives, etc.

Precision => x% of the time is correct vs incorrect.
When I click precision, it takes me to the images that are incorrectly classified.

In some cases, my label was wrong.
Is this the only suorce of error?

You may wonder, where should i start?  What to explore next?  Top level section shows ways to start.

Clicking correct shows me correct items.  Incorrect shows me incorrect items.
Consider adding more shape variations to traiing data for carrots, because my carrot is oddly-shaped.

Use finder to label things.

Case with multiple vegetables.  It says it's an eggplant, and it's true that there are eggplants here.  But other things as well.  need to think about if this is a usecase.  Perhaps the UI can guide my users to make sure to only point to one type of fruit at a time.  Consider usingo bject detection, another template in the app, rather than the whole image classifier.

Top confusion => Pepper as bean.  Maybe model is having trouble with peppers in general.most of my training data is bell pappers, make sure ihave good pepper representation in training data.

In a few minutes, the app helped me visually identify a few issues.  I need to make more tweaks to training data and see if it fixes the issue we saw.

All these explorations were possible because I had labeled data.  But how to test unlabeled examples?  Preview tab can help.

* Interactive evaluation
* Live preview
	* any attached webcam, continuity camera (ventura)
# Framework
Available on macOS, iOS, and iPadOS.  This year we're adding tvOS 16.

Programmatic interface not only lets you automate model creation and development time but also many new opportunities that learn directly from user input or on-device behavior providing personalized, adaptive experience.

Note that support does differ.
|         | macos | ios | tvOS |
| ------- | ----- | --- | ---- |
| tabular | yes   | yes | yes  |
| image   | yes   | yes | no   |
| sound   | yes   | yes | no   |
| text    | yes   | yes | no   |
| motion  | yes   | no  | no   |
| video   | yes   | no  | no     |

One common question you may have: what if I can't map my idea into these predefined tasks?

New member to create ML family: Create ML components
* ML building blocks
* Modify, combine, customize
[[Get to know Create ML Components]]
[[Compose advanced models with Create ML Components]]

Endless capabilities!

# Ex
Repetition counter.  

when combined with action classification, lets you cound and classify.

repetition counting is available as runtime API.  REquires no training data, can be just a few lines of code.  Implementation based on pre-trained model designed to be class-agnostic.  It's also applicable to a wide variety of full-body repetitive actions.

Check out sample code.

# Wrap up
* Interactive evaluation
* tvOS support
* Repetition counting
* Create ML components

https://developer.apple.com/documentation/createmlcomponents/counting_human_body_action_repetitions_in_a_live_video_feed
https://developer.apple.com/documentation/CreateML
