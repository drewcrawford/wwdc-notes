#researchkit

# Community updates
blah blah covid etc

researchandcare.org

# Onboarding updates
Custom text and images.

`ORKInstructionStep()`
set title, set detailText, set image

## Consent

Now can do consent in 1 step, whereas previously it was 2 steps?

```swift
ORKWebViewStep(identifier, html)
webViewStep.showSignatureAfterContent = true
```

## Request health permissions step
`ORKRequestPermissionsStep()`

# Survey enhancements
## Onboarding Survey
## `ORKForms`
Display errors in labels
## SES answer format
Scale-based questions

`ROKSESAnswerFormat`
## `ORKDontKnow` button

## Max character count label, clear button
`ORKAnswerFormat.textAnswerFormat()`

## `ORKReviewViewController`

# Active Tasks
## Hearing tasks
Now prompts the user to find a quiet location

## 3d models

`ORK3DModelStep` `ORKUSDZModelManager`

Basically, can create tasks that highlight parts of a model, or that prompt the user to tap the model.

Why create a model manager instead of adding functionality directly to the step?

You can subclass to implement various other features.  `ORKBioDigitalModelManager`.

## Front-facing camera
First, we create `ORKActiveStep` subclass

Override `+(Class) stepViewControllerClass`

...this appears to be an explanation of sample code or how they implemented this step themselves?


github.com/researchkit
researchandcare.org

