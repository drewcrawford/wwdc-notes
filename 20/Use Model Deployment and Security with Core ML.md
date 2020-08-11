#coreml

# Model deployment
* Independent of app
* Model collections
* Targeted deployments

## Independent development
In the past, you had to push app updates just to update your models.

## Model collections
Deployment keeps models from the collection together and atomically delivers them.

## adopting
* adopt API
* prepare model
* deploy model

```swift
MLModelCollection.beginAccessing(identifier: "MyCollection") {
[self] result in
	switch result {
	case .success(let collection)
		collection.entries["My model"]
	case .failure(let error):
		//network not available, etc.
	}
}
```

## Targeted deployments
As your app evolves, it might makes sense to make models for targeted populations
* Language code
* device class
* OS
* OS version
* region code
* app version

## Best practices
* Testing each model locally before deployment
* Have fallback options
* Define collections around features


# Model encryption
Encrypts *compiled* models.

Xcode associates encryption key with a team.

Build phases -> compile sources -> compiler flags

`--encrypt "$SRCROOT/HelloFlowers/Models/FlowerStylizer.mlmodelkey"`

How to load encrypted model?

We will deprecate the default initializer, and we recommend moving to async method because it lets you handle load errors.

Even though coreml needs network access to fetch the key the first time, it won't need it subsequently.

I'm not really sure why you would use this feature.

## summary
1.  create key with xcode
2.  Encrypt model
3.  Load model on the device

# Xcode updates
model summaries are now better
model preview
