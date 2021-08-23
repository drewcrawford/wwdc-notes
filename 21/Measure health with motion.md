# Tracking changes in walking
* Injury recovery
* assess risk of falling
* aging
* mortality

Last year, we introduced mobility metrics

[[Beyond counting steps]]

# Metrics
Six-minute walk, New recalibration API, better capture acute changes in health.

Also talk about Walking Steadiness.

* Estimates how far you can walk
* Walking steadiness: quality of walk

Both metrics provide a unique window.

# Six-minute Walk
Staple for clinicians in many fields
* Indicates cardiovascular and musculoskeletal health
* Captures walking endurance for low capacity individuals
* Measures acute changes with recalibration API

## Recalibration
Six-minute walk looks at activity and motion across a historical measurement window.  

Use a brand new metadata field `HKMetadataData...`

Recalibrate our estimate to use data only post surgery.  More accurate pre and post surgery comparison.

`HKHealthStore.recalibrateEstimates(...)`

1.  Choose a SampleType
	1.  `allowsRecalibrationForEstimates`
	2.  six-minute walk is the only option
2.  com.apple.developer.healthkit.calibrate-estimates entitlement
3.  Get authorized for reading and sharing

Consider
* Estimates already in HealthKit are not affected
* New estimates could take up to 14 days of data
* Recalibration is temporary

Tradeoff between longer window accuracy and acute changes.

# Walking steadiness
Track quality of movement.  

## Components
* Strength
* Endurance
* Balance
* Injury

## Elements
* Score (quality of movement)
* Classification (indicates fall-level risk)
* Notification (alerts high-risk individuals)

## Examples
1.  Walking steadiness decreases as they age
2.  Preserve fitness improves score
3.  Acute events can change quality

## Classification
* OK - no issues
* Low - compromised mobility, e.g. difficult hikes.  Early awareness
* Very low - compromised steadiness.  High risk of falling

## Requirements
1.  Pant pocket, jacket pocket, purse held closely.  Device held tightly to the center

## ex

* Score
	* `.appleWalkingSteadiness`
	* percentage
	* 0-1
	* Up to weekly
* classification
	* `HKAppleWalkingSteadinessClassification`
		* `.ok`, `.low`, `.veryLow`
		* convenience constructor for `sample.quantity`
* Notification
	* Category `.appleWalkingSteadinessEvent`
		* `.initialLow` and `.initialVeryLow` about a month after dropping into these
		* `.repeatLow` and `.repeatVeryLow` will remind users if they stay

## Best practices
* Enter height, weight, and age in HealthKit
* Enable first-party notifications (in healthkit?)
* Encourage walking with iPhone
* Check for walking speed samples
	* Two weeks with consistent samples is a good indicator
	* iPhone 8 and better

# Track aging
* Monitor disease
* Tailor workouts

Encourage you to explore how cardiovascular health and mobility can be used for rehabilitation or prehabilitiation applications.

Combination of these metrics can unlock a powerful experience.

# Best practices
* Maintain privacy and security safeguards
* Be transparent about what you are collecting
* Ensure users have control of their data

* https://developer.apple.com/documentation/healthkit/hkapplewalkingsteadinessclassification
* https://developer.apple.com/documentation/healthkit/hkcategorytypeidentifier/3747012-applewalkingsteadinessevent
* https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3747016-applewalkingsteadiness
* https://developer.apple.com/documentation/coremotion/cmfalldetectionmanager
* https://www.apple.com/healthcare/docs/site/Measuring_Walking_Quality_Through_iPhone_Mobility_Metrics.pdf
* https://www.apple.com/healthcare/docs/site/Using_Apple_Watch_to_Estimate_Six_Minute_Walk_Distance.pdf
* https://www.apple.com/healthcare/docs/site/Using_Apple_Watch_to_Estimate_Cardio_Fitness_with_VO2_max.pdf
* https://developer.apple.com/documentation/coremotion/monitoring_movement_disorders
* https://developer.apple.com/documentation/coremotion




