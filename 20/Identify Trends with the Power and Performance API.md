Actionable tools to identify areas

[[Improving Battery Life and Performance - 19]]

We have received many requests to be able to access the data programmatically.  Ability to consume this data in your own analytics and monitoring pipelines.

[[Expanding Automation with the App Store Connect API]] to learn more about ASC.

New API enables you to programmatically access xcode metrics and diagnostics.  Smart insights to identify your app's power/perf hotspots without additional work.

Data is only gathered from devices that have explicitly provided consent.

# Build powerful automation
Custom analytics
monitoring systems

# overview
* four new REST API resources

## application metrics and insights
* requires app id
* Gets aggregate metrics for recent versions
* includes smart insights

## aggregate metrics for specific app versions
* requires build ID
* Get aggregate metrics for a specific version

## top diagnostic signatures per app version
* requires app ID
* Groups common problems

[[Diagnose Performance Issues with the Xcode Organizer]]

## Diagnostic logs per signature
* requires diagnostic signature ID
* gets diagnostic logs for specific signature


# Metrics and smart insights
## aggregated metrics data
Aggregated by unique metrics and device
same as available in organizer
* battery
* launch
* disk writes

Available for all iPhone and iPad device groups

```
GET /v1/apps/{id}/perfPowerMetrics
```

## response
* `identifier`
* `unit`
* 50% 90% percentile
* up to 8 most recent versions

## smart insights
Flags important power and perf hotspots
* regressions
* trends

Details per insight
* analyzed versions
* insight summary string
* list of impacted population datasets

### ex
* `metric` type
* `summaryString`.  
* `populations` impacted percentiles and devices
* metadata such as latestversion, referenceVersions, etc.

## diagnostics data
* signatures to group similar problems
* Disk write signature
* root cause analysis by problem group
* Top signatures sorted by weight
* Use signature ID to get diagnostic logs

```
GET /v1/builds/{id}/diagnosticSignatures
```

response contains signature id, attributes, relationships.  `weight` signifies relative importance.

## diagnostic logs
contains anonymized diagnostic details from devices
* diagnostic metadata
* call stack trees

[[What's New in MetricKit]]

```
GET /v1/diagnosticSignatures/{id}/logs
```


# demo
Get JWT access token
We will release sample code later this year

# wrap up
* view and monitor aggregated metrics
* Highlight power and performance hotspots
* Deep dive into root causes

* new API for power/performance data
* programmatic access to xcode organizer
* customized data analytics/monitoring
* Out of the box smart insights to identify key trends and regressions

Try it out and send feedback

