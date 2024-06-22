

Discover how Xcode Cloud can adapt to your development needs. We'll show you how to streamline your workflows, automate testing and distribution with start conditions, custom aliases, custom scripts, webhooks, and the App Store Connect API.

# Essential workflow concepts
* environment -> environment variables.  xcode, macOS version
* start conditions -> when workflow runs.  source control events, brach / PR / git tag / schedule, etc.
* build actions -> build, run tests, analyze, or archive. 
* post actions -> notify, notarize, distribute

# Scale your workflows

For most cases, a simple workflow is all that's needed.  But workflows can be scaled to meet your needs.

xcode cloud workflow which starts when code is pushed to the main branch.  Build, test, deliver.  

Integration tests.  More time and resources than other tests, and can be affected by e.g. network conditions.  New in 15.1, you can configure manual start conditions.

Custom aliases, new in 15.3.  Using your alias in one or more workflows, they run with version of xcode / macOS specified in alias.  All workflows will run with updated values.

[[Customize your advanced Xcode cloud workflows]]

all environment variables are available, see docs.

### Custom Script - 10:02
```swift
#!/bin/sh

set -e

if [[ $CI_XCODEBUILD_ACTION == "test-without-building" && $CI_WORKFLOW_ID == "82D89C93-B69C-46B5-A794-A2BCFD3EE487" ]]
then
    curl https://example.com/health --fail
fi
```


# Connect other systems


See docs.

[[Meet Swift OpenAPI Generator]]



### App Store Connect API - Client Extension - 14:01
```swift
extension Client {
    func repoID(_ workflowID: String) async throws -> String {
        return try await ciWorkflowsGetInstance(
            path: .init(id: workflowID),
            query: .init(include: [.repository])
        ).ok.body.json.data.relationships!.repository!.data!.id
    }
    
    func branchID(_ repoID: String, name: String) async throws -> String {
        return try await scmRepositoriesGitReferencesGetToManyRelated(
            path: .init(id: repoID)
        )
        .ok.body.json.data
        .filter { $0.attributes!.kind == .BRANCH && $0.attributes!.name == name }
        .first!.id
    }
    
    func startBuild(_ workflowID: String, gitReferenceID: String) async throws {
        _ = try await ciBuildRunsCreateInstance(
            body: .json(.init(
                data: .init(
                    _type: .ciBuildRuns,
                    relationships: .init(
                        workflow: .init(data: .init(
                            _type: .ciWorkflows,
                            id: workflowID
                        )),
                        sourceBranchOrTag: .init(data: .init(
                            _type: .scmGitReferences,
                            id: gitReferenceID
                        ))
                    )
                )
            ))
        ).created
    }
}
```



### App Store Connect API - Main Function - 14:43
```swift
static func main() async throws {
    let client = try Client(
        serverURL: Servers.server1(),
        configuration: .init(dateTranscoder: .iso8601WithFractionalSeconds),
        transport: URLSessionTransport(),
        middlewares: [AuthMiddleware(token: ProcessInfo.processInfo.environment["TOKEN"]!)]
    )
    
    let workflowID = "82D89C93-B69C-46B5-A794-A2BCFD3EE487"
    let repoID = try await client.repoID(workflowID)
    
    let branchName = "main"
    let branchID = try await client.branchID(repoID, name: branchName)
    
    try await client.startBuild(workflowID, gitReferenceID: branchID)
}
```

Builds started by ASC API are considered manual.

### Webhook Handler Implementation - 17:09
```swift
struct WebhookPayload: Content {
    let ciWorkflow: CiWorkflow
    let ciBuildRun: CiBuildRun
    
    struct CiWorkflow: Content {
        let id: String
    }
    
    struct CiBuildRun: Content {
        let id: String
        let executionProgress: String
        let completionStatus: String
    }
}

func routes(_ app: Application) throws {
    let deploymentService = ExampleDeploymentClient()
    let workflowID = "82D89C93-B69C-46B5-A794-A2BCFD3EE487"
    
    app.post("webhook") { req async throws -> HTTPStatus in
        return HTTPStatus.ok
    }
}
```

# Wrap up
* configure manual start conditions, custom aliases, and custom scripts
* Integrate ASC API and webhooks
[[Create practical workflows in Xcode Cloud]]
[[Meet Xcode Cloud]]

# Resources
* [Configuring start conditions](https://developer.apple.com/documentation/Xcode/Configuring-Start-Conditions)
* [Configuring webhooks in Xcode Cloud](https://developer.apple.com/documentation/Xcode/Configuring-Webhooks-in-Xcode-Cloud)
* [Environment variable reference](https://developer.apple.com/documentation/Xcode/Environment-Variable-Reference)
* [Forum: Developer Tools & Services](https://developer.apple.com/forums/topics/developer-tools-and-services?cid=vf-a-0010)
* [Sharing build configurations across Xcode Cloud workflows](https://developer.apple.com/documentation/Xcode/Sharing-build-configurations-across-xcode-cloud-workflows)
* [Writing custom build scripts](https://developer.apple.com/documentation/Xcode/Writing-Custom-Build-Scripts)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10200/5/28E5AAA4-9AE8-427A-B577-512070861A1A/downloads/wwdc2024-10200_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10200/5/28E5AAA4-9AE8-427A-B577-512070861A1A/downloads/wwdc2024-10200_sd.mp4?dl=1)