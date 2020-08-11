#healthkit 
Users access data from various devices and apps.
We never want to surprise users.

**Empower users**.  Changes to data must reflect user's intent.

# Monitoring changes in HealthKit
Apps must be equipped to react appropriately to healthkit changes.

Lets patients track their daily step count.  Graph data per day.  Tracked on multiple devices.

`HKStatisticsCollectionQuery` -> first tool you should reach for.

[[Getting Started with HealthKit]]

Chart data sent to server.  New statistics sent to server as well.  
Other situations with "external data" might include an in-app database for example.

## Re-querying with `HKStatisticsCollectionQuery`
* Performs redundant calculations
* Waste of network resources

## `HKAnchoredObjectQuery`
* monitor updates to the health database
* provides a snapshot of changes
* Includes new sampels and deleted samples

Anchor -> a specific point in time in th evolution of the database.  Identify samples you last received from this query.  

Initially use anchor `nil`, then re-use the last anchor.  We give you new samples since the last anchor.

## Querying for minimal data
* think about the type of data.  e.g., cumulative statistics vs samples.
* Use case drives type of query.
* Better performance
## example
Not copying this code block
or the next one

# Importing external data
Write walk test results to healthkit.

Report contains a graph of the 6-minute walk-test distance.  Also individual samples displayed underneath.

In this case, we're interested in the individual samples.  So it's worth logging each sample, not a combined value.

## making changes to healthkit
Add incremental samples.  Removing/reinserting data can be unexpected.
New samples represent new data.

Be careful when deleting samples.  

Reflect user intent.  User might not mean for you to delete the sample.

## challenges in syncing
* Need to delete, then insert, to avoid duplicate data.  May need to query first.
* Avoid duplciate samples
* Samples exist on all devices.  Change in samples need to be reflected correctly on all devices.  Ensure that you aren't saving the sample on both devices.

## `HKmetadataKeySyncIdentifier` and `HKMetadataKeySincVersion`

ID allows us to recognize sample across devices
Version helps us understand when the sample has been updated.

By setting sync identifier, HK does deduplication.
HK only updates samples when the version number has increased.
If there's any error, you can be rest-assured that your data is in a consistent state
Multi-device consistency

Now when making changes, bump version number.  HK overwrites prior samples with new samples.

## Consider the following architecture
Report
contains 6 samples
each sample points to an ID like `Week1SampleN`

Create a sample from each thing in server response.
Add ID/version metadata.
save to healthkit.


# Wrap up
* reflect the user intent
* Keep users in control of their data
* Run efficient queries
* Use sync identifiers and version numbers

# Next steps
* think about security and privacy
* CareKit for visualizations
* Many other HealthKit features



# Resources
https://developer.apple.com/documentation/healthkit/creating_a_mobility_health_app
https://developer.apple.com/documentation/healthkit
