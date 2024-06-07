Apple devices can connect to multiple networks at the same time. Learn how your app can automatically select the best one for an optimal experience. Explore the different types of networks and review their characteristics. And discover how to use URLSession and Network framework to best describe your needs, so the system can intelligently choose the best interface for your app at all times.

# Overview

Cellular, wifi.  Dual band.  Multipath TCP, QUIC.  

bulk data.  cellular, personal hotspot, low data mode, etc.
low data use.
ethernet connection.
# Describe your needs

**Reason about the properties of the network, rather than the type of the network**.

Constrained networks.  Apps use less data.  
`allowsConstrainedNetworkAccess` and `prohibitsContainedPath` -> NetworkFramework.

Limit transfers to small payloads and only perform tasks that enable the user's explicit actions.

Expensive.  Only used for user-initiated work.  `allowsExpensiveNetworkAccess` or `prohibitExpensivePaths`.

Consider using the same logic for both cases!

Just try to connect.  
Avoid network checks before requests.
Network conditions change frequently, and changes can be triggered by app's request.  Preflight checks are incorrect.
Reachability APIs are deprecated.
Instead, try to connect.  
`URLSessionConfiguration.waitsForConnectivity`

# Know when to try again

System waits for connectivity `taskIsWaitingForConnectivity`
communicate the reason
provide a wayt o continue
use alerts sparingly

# Test your app

Simulate different network conditions with xcode.  Devices/simulators.  ex, network liink condition and LTE profile to simulate a typical LTE network in average conditions.  Test workflows to ensure everything is smooth and fluid.

Network link conditioner.  

# next steps
describe networing needs
replace constriants for specific interfaces
Remove preflight checks in your app
avoid rertries, wait for connectivity instead


[[Accelerate networking with HTTP3 and QUIC]]


