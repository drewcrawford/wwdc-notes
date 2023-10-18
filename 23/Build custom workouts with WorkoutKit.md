WorkoutKit makes it easy to create, preview, and schedule planned workouts for the Workout app on Apple Watch. Learn how to build custom intervals, create alerts, and use the built-in preview UI to send your own workout routines to Apple Watch.

How to build custom workouts for apple watch.  In OS9, we introduced new workout types.

goal-based workouts: perform workout with a singular goal, such as distance, time, etc.
pace: pace/speed front and center
swim/bike, etc.
custom workouts: structured steps with a combination of custom goals and alerts.

In iOS17, watchOS10, we're bringin these into WorkoutKit #workoutkit

brand new swift framework
supports all workout types
preview UI
sync scheduled workouts

today we'll focus on custom workouts.

# Building a custom workout
great way for users to focus their workout in a structured manner.  Distinct steps.  3 stages.
* warmup
* ordered collection of repeatable blocks, combination of steps.  Represent the majority of the workout
* cooldown

every step contains two important attributes.
1.  single goal.  Define the progression of steps within a custom workout.  When the goal is complete, we move to the next step.
	2. time or distance goal.  Or open goal, requires the user to manually progress.
3. alert.  ex, the user may want to be alerted when their heartrate is elevated past a certain threshold.  Pace, cadence, power, heartrate alerts.

blocks.

Blocks contain steps distinguished as work steps or recovery steps.  any number of steps in any order within a block.  Blocks can also be repeatable.  Specify the number of iterations, etc.

demo

with steps and blocks, you can construct a complete custom workout.

demo

warmup - open goal
block - 4 iterations
block - 2 iterations
cooldown - time goal

code samples not provided

our first block is complete.  let's move on to our second block.

CustomWorkoutComposition is the megazord that assembles all our steps.

note that we validate the composition which is why there' s an error

## validations
* prevent issues during workout runtime
	* distance goals should not be used for non-distance-based workout positions
	* no pace for elliptical
	* etc
* run throughout workout APIs

## composition serialization
* use workoutcomposition wrapper
* serialize workout into two formats
* export as `.workout` to be recognized by the system.  binary is smaller than json!

# previewing for export
* export to `.workout` file
	* we validate during export
* preview options
	* iOS - an out of process UI is presented with the workout composition.  Option to save.
	* watchOS - launches the app with the contents of the composition.  User is able to immediately start the workout or save for later. 
use any workout type mentioned earlier.  Let's present the preview .




# Scheduling workouts

What if you have a collection of workouts?  Ex, let’s say you have some cycling scheduled.  Later you want to do hiking.  Then golfing, rest, more cycling, etc.

Now the user is responsible for managing all these workouts and remembering when they need to complete them.  Instead, schedule workouts directly into the workout app.

Your app will have a dedicated space at the top of the app.  Styled with app icon and name

Preview of the next workout for the day.  Tapping the ellipsis will show more details, etc.

* secure and private
* Visible for seven days before and after today
* Supports up to 50 workouts
* Query for completed scheduled workouts
	* Only contain composition, date, and whether completed.  No health data.
	* Refer to HealthKit for health statistics.
Retrieve a workout composition from an HKWorkout.

Set of apis to suport syncing compositions to Apple Watch.

Get workoutplan.authorizationState.  

Now that we have authorized our app, get our current workout plan, etc.

Interface to store and modify scheduled workouts from our app.

The completion state will be updated.  If a workout is marked as completed update the completion state ot make sure the user has the most recent information.

Keep your app in sync with the owrkout app.  

## best practices
* consider all composition types
* swimming isn’t supported with custom workouts, need to use a goal composition isntead
* alerts only available for custom workouts.  If you don’t need them, consider using standard goal
* Handle validations
* Keep scheduled workouts up to date
* Send feedback
* 

* https://developer.apple.com/documentation/healthkit
* https://developer.apple.com/documentation/Updates/watchos
* https://developer.apple.com/documentation/workoutkit
