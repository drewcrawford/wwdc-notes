#healthkit

\>18k applications on the app store that focus on improving health and fitness

Creates a central repository of user's health data.  Securely stores/synchronizes accross devices.  Guards privacy.

# Smoothwalker
Record walks, visually see the daily steps.

Accomplish different tasks.

## Setup HealthKit
1.  Give your application access.  Add the capability.
2.  Check platform availability

```swift
if HKHealthStore.isHealthDataAvailable() {
	/* */
}
else {
	/* */
}
```

3.  Create an `HKHealthStore`

## `HKHealthStore`
*Your gateway to the world of health*
* ask for authorization
* write health data
* fetch data via queries
* create once and reuse


## Save walking distance
Save walking distance.  Before we do, we'll need to take a deeper look into how HK organizes and structures our data.

### Quantity samples
Numerical value and unit.  

### authorization
* Authorization granted by type
* Read and write are separate
* May not get all permissions requested

#### best practices
* ask in context
* "Health update usage description"
* "Health share usage description"
* Timing is key.  
* Ask for only what you need
* Ask every time

### save walking distance
1.  Request write access

```swift
let distanceType = HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
healthStore?.requestAuthorization(toShare: [distanceType], read: nil) { success, error in
	if success {
		//note that this doesn't mean you got permission, just that you successfully *requested authorization*.
	}
	else {
		//failed to request, not denied authorization
	}
}
```

**Only what you need, when you need it, every time**

2.  Create the sample
3.  Save sample to healthkit

```swift
let distanceType = HKSampleType.quanittyType(forIdentifier: .distanceWalkingRunning)!

let startDate = ...
let endDate = ...

//construct a quantity, representing the value and unit for the distance walked
let distanceQuanitity = HKQuantity(unit: HKUnit.meter(), doubleValue: 628.0)
let sample = HKQuantitySample(type: distanceType, quanitty: distanceQuantity, start: startDate, end: endDate)

healthStore.save(sample) { success, error in
	if success {

	}
	else {
	}
}
```

## healthkit sample types
* `HKQuantitySample` -> numerical data and unit.
* `HKCategorySample` -> Value you report comes from a preassigned list, no unit.  ex. mild, moderate, severe
* `HKWorkout` -> Summarizes multiple values, multiple units for multiple values
* many more!

* Start and end time

## characteristics
Birthday, blood type.  Static.

## `HKObject`
* unique identifier
* app
* device

## Types
HK has a parallel "NSClass"-like system to represent the type of each object (as a value).  `HKObjectType`, `HKSampleType`, etc.


## display daily steps

Use queries.

* construct a query object
* type of data
* predicate
* Execute on `HKHealthStore`
* Results delivered in handler



## `HKStatisticsQuery`
* Sum: total steps, total calories, total caffeine.  Cumulative.
* average (SPF, heart rate, body temperature).  Also min/max.  Discrete.

Note that these are kinda mutually exclusive.  We call this dichotomy the "aggregation style of samples".

Note that stuff like "steps" is really complicated.  Maybe you were walking with a watch and iphone, and they both recorded steps.  `HKStatisticsQuery` knows how to deduplicate this.  Vs, walking one without the other, and `HKStatisticsQuery` can grab both, not just from the device you're on now.

## `HKStatisticsCollectionQuery`
Performs statistics on fixed time intervals that you specify.  e.g., per day.
* anchor date - when statistics start being computed.
* time interval - cadence
* statistics to compute: sum

This query tye can receive ongoing updates.
* Set update handler before executing
* Query listens to new statistics.
* Runs idefinitely.  Call `stop` when we're done collecting data.

## display daily steps

1. Construct the query
2. Execute
3.  Update our UI with results

## demo
https://developer.apple.com/documentation/healthkit/creating_a_mobility_health_app

# `HKSampleQuery`
Can retrieve raw data stored in the database.

```swift
let workoutType = HKObjectType.workoutType()
let sort = [NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)]

let query = HKSampleQuery(sampletype: workoutType!, predicate: nil, limit: 1, sortDescriptors: sort)
{
	// ...
}
healthStore.execute(query)
```

# Other query types
* `HKAnchoredObjectQuery` -> detect changes in the user's database
* `HKActivitySummaryQuery` -> display activity ring data
* `HKWorkoutRouteQuery` -> display locations we've been during outdoor workouts

# Takeaways
* Healthkit central repository of user's data
* Health data is very sensitive.  Request authorization for only the data you need, when you need it, every time.
* Rich typesystem to organize and structure health data.  Many more than we covered.
* Wide range of queries.  Many more 

# Resources
https://developer.apple.com/documentation/healthkit/creating_a_mobility_health_app
https://developer.apple.com/documentation/healthkit/samples/accessing_health_records
https://developer.apple.com/documentation/healthkit


