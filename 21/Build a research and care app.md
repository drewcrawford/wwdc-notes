#researchkit 
#carekit 

Recover a physical therapy app to help people strengthen their knees following surgery.

# Onboarding and consent

Onboarding and consent is important to a good study.  New best practices to collect this info.

1.  Gate participation
2.  Present instructions
3.  Require signature
4.  Request permissions

Persist an onboarding task.
Define a schedule taht specifies how often it will appear.  For onboarding we use a daily schedule.

ONboarding should *not* impact adherence.  Doesn't count toward completion rings.

The next step is to setup that onboarding flow.

Onboarding survey has 5 steps
1.  Welcome
	1.  ORKInstructionStep.  
2.  Instruction
	1.  Instruction step.  Body items.  
3.  Signature
	1.  ORKWebViewStep
	2.  showSignatureAfterContent = true
4.  Permissions
	1.  ORKHealthKitPermissionType
	2.  ORKNotificationPermissionType
	3.  ORKMOtionActivityPermissionType
5.  completion
	1.  ORKCompletionStep

Chain everything together inside ORKOrderedTask

OCKSurveytaskViewController.  Converts researchkit result into carekit outcome values.

Ensure feed reloads once participant finishes onboarding.  Set ourselves as survey task VC delegate.  Implement method `didFinish result:`.  

## Demo

# Schedule tasks
* Display multi-question forms
* Persist data with CareKit
* Create dynamic schedules
* Measure range of motion

1.  Create researchkit survey
2.  researchkit takes over and does whatever
3.  result returned to us
4.  convert to CK outcome
5.  store

non-optional question => can't be skipped
format => what kind of answer to expect, UI to enter it, etc.

ORKTaskResults are nested types.  Start at the root result and drill in.  

## Demo

## Create dynamic schedules
Reduce how often we ask participants as time goes on.  Every day for the first week, but only once a week until the end of the month.  Then never.

OCKScheduleElement.  Start date, end date, repeat at some interval.

Range of motion task - built in.  Omit its completion step and define our own.

## Demo

# Visualize progress
* Enhance surveys
* Display charts
* Show 3D model of knee

Since there are no results or schedule for showing the model, won't need to integrate with CareKit schedules or tasks.

Investigator grants, active on GitHub, etc.

# Wrap up
* https://github.com/carekit-apple/WWDC21-RecoverApp.git
* https://www.researchandcare.org
* https://developer.apple.com/documentation/healthkit


