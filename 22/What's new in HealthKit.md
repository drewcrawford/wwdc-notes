#healthkit 

# Sleep analysis
Apple watch will automatically track all the different sleep stages you go through when you're asleep.  This data willb e accessible from the app and saved in HK.

`HKCategoryType.sleepanalysis`
3 sleep stages
* REM
* Core
* Deep
1 sample for each time period in a given sleep stage

Create a predicate for sleep samples in a given stage.  If you want all samples, ensure you query for `.allAsleepValues`.

Since iOS 15.4, we updated our query api to support swift async.
# Swift async
Subclasses of HKQuery. `HKSTatisticsCollectionQuery`.  Previously, you used completion handlers.  Thansk to swift async, we're making this simpler.

If I want live updates, it's as simple as calling `results(for:)` and looping through the returned async sequence.


# Workouts
New workouts: swim, bike, run (triathlon?)
Each activity is represented by HKWorkoutActivity.

workout activity holds a list of events.

read statistics for each activity.

Activities aer not required to be continuous.  Create an activity for each transition.  

1.  Create a workout configuration with type `.swimBikeRun`
2. Create workout session
3. startActivity
4. associatedWorkoutBuilder.beginCollection
5. beginNewActivity
6. enableCollection
7. endCurrentActivity
8. beginNewActivity
9. disableCollection(prior activity)
10. end session (ends all activities)
11. workoutBuilder.finishWorkout()
12. Read metrics from workout
	13. old methods deprecated
	14. use `statistics(for: HKQuantityType)`
	15. Automatically computed from samples collected during the workout

New predicates to query for workouts you're interested in.  

query high intesnity worktouts
1.  create predicate
2. wrap inside workout predicate
3. create query inside taht predicate
4. result(for healthstore)

workout intervals => make sure all activities are configured with the same type.
You can get statistics for each interval.  

New running metrics that are automatically collected on s6, SE, and newer.

For swimming metrics, we 're adidng SWOLF score.  Strokes per length + time per length
Each lap event and segment event on apple watch.

heart rate recovery.  Estimate of how quickly your heart rate lowers after exercise.  Can be used to understand how heart recovers after stress and reveal potential health problems.

We're introducing a new cardio recovery tab.  Read, save this data.
* Quantity type
* `.heartRateRecoveryOneMinute`
* additional context in metadata

When using HKLiveWorkoutBuilder, a heartkit recovery sampel is automatically built/saved.  Otherwise, you can save manually with HKQuantitySample(type:)
test type: max exercise?  Or whatever makes sense
duration, activity type, observed recovery heart rate, etc.

New metrics provides a more comprehensive picture of your workouts.
# Vision prescriptions
In fact, according to vision council, 75% of us adults rely on vision correction.
Easy to lose, one more thing you need to have with you, etc.  Starting with iOS 16, your apps can save glasses and contacts prescriptiosn in healthkit

HKSampletype.visionPrescriptionType()
Sample date range = validity range
Physical prescription as an attachment

classes
HKVisionPrescription (abstract)
HKGlassesPrescription (subclass)
HKContactsPrescription (subclass)

Two HKGlassesLensSpecification, one for each eye

also HKContactsLensSpecification, one for each eye.

File attachment, HKAttachment.  HKAttachmentStore to save and read files.  Static image or pdf file.  

Prescription contains a lot more information than just a lens specification.  Full name, dob, etc.  One of HK's core principles is to protect privacy.

New authorization model for prescriptions.
* granted for each object separately
* New API to request object authorization
* Use queries to read prescriptions

Request authorization to read prescriptions
`.requestPerObjectReadAuthorization(for:predicate:)`.

Displays list of all prescriptions that match your predicate.  




* https://developer.apple.com/forums/tags/wwdc2022-10005
* https://developer.apple.com/forums/create/question?&tag1=121&tag2=299&tag3=436030
* https://developer.apple.com/design/human-interface-guidelines/healthkit/overview/
* https://developer.apple.com/documentation/healthkit