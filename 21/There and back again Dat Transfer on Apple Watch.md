#watchOS 

Independent watch apps

# Pick the right API?
* Type of data
* Data source and destination
* Reliance on companion iOS app
* Support Family Setup
* Timing


# Keychain with iCloud Synchronization
#keychain

* Password autofill
* Shared keychain items

## Password autofill
Add the Associated Domains capability to your targets
(watchkit extension target)
Add webcredentials entry
Add `apple-app-site-association` file

Associate both app and extension in the `apps` array

Add textContentTYpe modifiers to your TextFields
```swift
struct LoginView: View {
    
    @State private var username = ""
    @State private var password = ""
    
    var body: some View {
        Form {
            TextField("User:", text: $username)
                .textContentType(.username)
            
            SecureField("Password", text: $password) 
                .textContentType(.password)
            
            Button {
                processLogin()
            } label: {
                Text("Login")
            }
            
            Button(role: .cancel) {
                cancelLogin()
            } label: {
                Label("Cancel", systemImage: "xmark.circle")
            }
        }
    }
    
    private func cancelLogin() {
        // Implement your cancel logic here
    }
    
    private func processLogin() {
        // Implement your login logic here
    }
}
```

Suggestions availble since 6.2

[[AutoFill Everywhere]]
## Shared Keychain items
* Secure storage
* Small bits of data
* Information that changes infrequently
* Synchronized to all devices

* Add Keychain Sharing App Groups Capability to targets
* Add apps to common Keychain Group or App Group

```swift
func storeToken(_ token: OAuth2Token, for server: String, account: String) throws {
    let query: [String: Any] = [
      kSecClass as String: kSecClassInternetPassword,
      kSecAttrServer as String: server,
      kSecAttrAccount as String: account,
      kSecAttrSynchronizable as String: true,
    ]
    
    let tokenData = try encodeToken(token)
    let attributes: [String: Any] = [kSecValueData as String: tokenData]
    
    let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
    
    guard status != errSecItemNotFound else {
        try addTokenData(tokenData, for: server, account: account)
        return
    }
    
    guard status == errSecSuccess else {
        throw OAuthKeychainError.updateError(status)
    }
}
```

important to include `kSecAttrSynchronizable`.

Retrieve token

```swift
func retrieveToken(for server: String, account: String) throws -> OAuth2Token? {
    let query: [String: Any] = [
      kSecClass as String: kSecClassInternetPassword,
      kSecAttrServer as String: server,
      kSecAttrAccount as String: account,
      kSecAttrSynchronizable as String: true,
      kSecReturnAttributes as String: false,
      kSecReturnData as String: true,
    ]
        
    var item: CFTypeRef?
    let status = SecItemCopyMatching(query as CFDictionary,
                                     &item)
        
    guard status != errSecItemNotFound else {
        // No token stored for this server account combination.
        return nil
    }
    
    guard status == errSecSuccess else {
        throw OAuthKeychainError.retrievalError(status)
    }
    
    guard let existingItem = item as? [String : Any] else {
        throw OAuthKeychainError.invalidKeychainItemFormat
    }
    
    guard let tokenData = existingItem[kSecValueData as String] as? Data else {
        throw OAuthKeychainError.missingTokenDataFromKeychainItem
    }
    
    do {
        return try JSONDecoder().decode(OAuth2Token.self, from: tokenData)
    } catch {
        throw OAuthKeychainError.tokenDecodingError(error.localizedDescription)
    }
}
```

Remove token
```swift
func removeToken(for server: String, account: String) throws {
    let query: [String: Any] = [
      kSecClass as String: kSecClassInternetPassword,
      kSecAttrServer as String: server,
      kSecAttrAccount as String: account,
      kSecAttrSynchronizable as String: true,
    ]
  
    let status = SecItemDelete(query as CFDictionary)
    
    guard status == errSecSuccess || status == errSecItemNotFound else {
        throw OAuthKeychainError.deleteError(status)
    }
}
```
Be aware that customers can disable and that it isn't available in all regions.
# CoreData with CloudKit
#coredata 
Think carefully about how much data you really want on the watch.  Consider using multiple configurations to segment data that makes sense in watch vs app with mroe storage and battery.

```swift
import CoreData
import SwiftUI

struct CoreDataView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    
    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Setting.itemKey, ascending: true)],
        animation: .easeIn)
    private var settings: FetchedResults<Setting>
    
    var body: some View {
        List {
            ForEach(settings) { setting in
                SettingRow(setting)
            }
        }
    }
}
```
Don't expect it to be instantaneous.

[[Build apps that share data through CloudKit and Core Data]]
[[Bring Core Data concurrency to Swift and SwiftUI]]

# Watch Connectivity

#watchconnectivity
* Communication between paired Watch and iPhone
* Share data only available on either Watch or iPhone

## Tips
* Activate WCSession as early as possible in your app lifecycle
* App not required to be reachable.  Understand reachability
* All WCSession delegates are called on a non-main serial queue

## Application context
* Property list dictionary
* Sent in the backgroudn to be available when the counterpart wakes up
* Previous value is replaced with new data
* `updateApplicationContext:` can be called when counterpart is not reachable

Useful for keeping content up to date on the counterpart app.

## User info transfer
* property list dictionary
* Sent sequentially in the background
* Use `outstandingUserInfoTransfers` to access and cancel unsent data

## File transfer
* Sent in the background (power etc. permits)
* Use `outstandingFileTransfers` to access and cancel unset files
* Received files are stored in the inbox directory
* Files are deleted from the inbox after delegate `session:didReceiveFile:` callbakc function returns
	* Move or process file before you return!
	* *If you call an async method, you will most likely run into a problem becuase the file will be gone*
	* Larger files take longer

## Current complication user info
* Send complication-related data to watch
* Update up to 50 times/day with an active Complication
* Sent as soon as possible when budget and connectivity allows
* Check budget with `remainingComplicationUserInfoTransfers`
* Sent as normal user info transfer when no budget remains

## Send message
* Interactive messages to the counterpart app
* sEnd a property list dictionary or data and get a reply
* counterpart must be reachable
* Keep messages small
* `replyHandler` is recommended
* Implement delegate method `didReceiveMessage:` or `didReceiveData:` with `replyHandler` in counterpart app
## Reachability
* Check the `WCSession.isReachable` property to determine reachability

What does it mean?
* Devices are in bluetooth or Wi-Fi range
* WatchKit extension: Running in the foreground or in the background (high priority bg only).  ex, background sessions.
* iOS app: Can be activated in background.  So iOS is reachable from WE more often than vice versa.

[[Introducing Watch Connectivity - 15]]
# URL Sessions
#urlsession
For servers, best option is URL sessions.
* Background URL sessions
* Foreground URL sessions

## Background
* Any time communication can be delayed
* Foreground sessions need to complete while your app is in the foreground or frontmost.  For the shortest tasks, this isn't enough time.
	* Be considerate and evaluate each task
* All alrge data transfers
* Updates initiated by a server
* Timing depends on system conditions

```swift
class BackgroundURLSession: NSObject, ObservableObject, Identifiable {
    private let sessionIDPrefix = "com.example.backgroundURLSessionID."
    
    enum Status {
        case notStarted
        case queued
        case inProgress(Double)
        case completed
        case failed(Error)
    }
    
    private var url: URL

    /// Data to send with the URL request.
    ///
    /// If this is set, the HTTP method for the request will be POST
    var body: Data?
    
    /// Optional content type for the URL request
    var contentType: String?
    
    private(set) var id = UUID()
    
    /// The current status of the session
    @Published var status = Status.notStarted
    
    /// The downloaded data (populated when status == .completed)
    @Published var downloadedURL: URL?
    
    private var backgroundTasks = [WKURLSessionRefreshBackgroundTask]()
    
    private lazy var urlSession: URLSession = {
        let config = URLSessionConfiguration.background(withIdentifier: sessionID)
            // Set isDiscretionary = true if you are sending or receiving large 
            // amounts of data. Let Watch users know that their transfers might 
            // not start until they are connected to Wi-Fi and power.
            config.isDiscretionary = false
            config.sessionSendsLaunchEvents = true
            return URLSession(configuration: config,
                              delegate: self, delegateQueue: nil)
        }()
    
    private var sessionID: String {
        "\(sessionIDPrefix)\(id.uuidString)"
    }
    
    /// Initialize the session
    /// - Parameter url: The URL for the Background URL Request
    init(url: URL) {
        self.url = url
        super.init()
    }

}
```

```swift
// This is a member of the BackgroundURLSession class in the example. 
// Enqueue the URLRequest to send in the background. 
func enqueueTransfer() {
    var request = URLRequest(url: url)
    request.httpBody = body
    if body != nil {
        request.httpMethod = "POST"
    }
    if let contentType = contentType {
        request.setValue(contentType, forHTTPHeaderField: "Content-type")
    }
    let task = urlSession.downloadTask(with: request)
    task.earliestBeginDate = nextTaskStartDate
  
    BackgroundURLSessions.sharedInstance().sessions[sessionID] = self
  
    task.resume()
    status = .queued
}
```

```swift
class ExtensionDelegate: NSObject, WKExtensionDelegate {
    
    func applicationDidFinishLaunching() {
        // For Watch Connectivity, activate your WCSession as early as possible
        WatchConnectivityModel.shared.activateSession()
    }
    
    func applicationDidBecomeActive() {
        // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    }
    
    func applicationWillResignActive() {
        // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
        // Use this method to pause ongoing tasks, disable timers, etc.
    }
    
    func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
        // Sent when the system needs to launch the application in the background to process tasks. Tasks arrive in a set, so loop through and process each one.
        for task in backgroundTasks {
            // Use a switch statement to check the task type
            switch task {
            case let backgroundTask as WKApplicationRefreshBackgroundTask:
                // Be sure to complete the background task once you’re done.
                backgroundTask.setTaskCompletedWithSnapshot(false)
            case let snapshotTask as WKSnapshotRefreshBackgroundTask:
                // Snapshot tasks have a unique completion call, make sure to set your expiration date
                snapshotTask.setTaskCompleted(restoredDefaultState: true, estimatedSnapshotExpiration: Date.distantFuture, userInfo: nil)
            case let connectivityTask as WKWatchConnectivityRefreshBackgroundTask:
                // Be sure to complete the connectivity task once you’re done.
                connectivityTask.setTaskCompletedWithSnapshot(false)
            case let urlSessionTask as WKURLSessionRefreshBackgroundTask:
                if let session = BackgroundURLSessions.sharedInstance()
                        .sessions[urlSessionTask.sessionIdentifier] {
                    session.addBackgroundRefreshTask(urlSessionTask)
                } else {
                    // There is no model for this session, just set it complete
                    urlSessionTask.setTaskCompletedWithSnapshot(false)
                }
            case let relevantShortcutTask as WKRelevantShortcutRefreshBackgroundTask:
                // Be sure to complete the relevant-shortcut task once you're done.
                relevantShortcutTask.setTaskCompletedWithSnapshot(false)
            case let intentDidRunTask as WKIntentDidRunRefreshBackgroundTask:
                // Be sure to complete the intent-did-run task once you're done.
                intentDidRunTask.setTaskCompletedWithSnapshot(false)
            default:
                // make sure to complete unhandled task types
                task.setTaskCompletedWithSnapshot(false)
            }
        }
    }
}
```
Connect extension delegate to app
```swift
@main
struct MyWatchApp: App {
    
    @WKExtensionDelegateAdaptor(ExtensionDelegate.self) var extensionDelegate
    
    @SceneBuilder var body: some Scene {
        WindowGroup {
            NavigationView {
                ContentView()
            }
        }
    }
}
```
```swift
// This is a member of the BackgroundURLSession class in the example. 
// Add the Background Refresh Task to the list so it can be set to completed when the URL task is done.
func addBackgroundRefreshTask(_ task: WKURLSessionRefreshBackgroundTask) {
    backgroundTasks.append(task)
}
```

```swift
extension BackgroundURLSession : URLSessionDownloadDelegate {
    
    private func saveDownloadedData(_ downloadedURL: URL) {
        // Move or quickly process this file before you return from this function.
        // The file is in a temporary location and will be deleted.
    }
    
    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {
        saveDownloadedData(location)
      
        // We don't need more updates on this session, so let it go.
        BackgroundURLSessions.sharedInstance().sessions[sessionID] = nil
      
        DispatchQueue.main.async {
            self.status = .completed
        }
        
        for task in backgroundTasks {
            task.setTaskCompletedWithSnapshot(false)
        }
    }
}
```
## Foreground
* Quick server communication
* Immediate data during app interaction
* 2.5 minute timeout **is ENFORCED**

## summary

[[Keep your complications up to date]]
[[Background execution demystified]]

# Sockets
> If you're building a streaming audio app

* HLS
* Web Sockets

[[Streaming audio on watchos 6]]

|                                 | Source, Destination     | Relies on companion iPhone | Supports Family Setup | Best For                   |
|---------------------------------|-------------------------|----------------------------|-----------------------|----------------------------|
| iCloud Keychain synchronization | All devices             | No                         | Yes                   | Infrequently changing data |
| Core Data with CloudKit         | All devices and iCloud  | No                         | Yes                   | Structured data            |
| Watch connectivity              | Paired iPhone and Watch | Yes                        | No                    | Optimization               |
| URL Sessions                    | Server                  | No                         | Yes                   | Most server communication  |
| Sockets                         | Server                  | No                         | Yes                   | Streaming audio            |

# Wrap up
* Think about data, source and destination, and customers before selecting a solution
* Pick the right tool for the job
* Test on device before deployment

* https://developer.apple.com/documentation/clockkit/keeping_your_complications_up_to_date
* https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_from_websites
* https://developer.apple.com/documentation/watchconnectivity/wcsession
* https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps
* https://developer.apple.com/documentation/watchkit/keeping_your_watchos_content_up_to_date
* https://developer.apple.com/documentation/xcode/supporting-associated-domains

