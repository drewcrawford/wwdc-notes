#networking 

# Evolution of HTTP
1.  Connection setup
2.  GET
3.  idle
4.  200 OK

Have to go through the same process again.

A lot of time is spent on connection setup.  What if we reuse a single connection?  Save setup time but reques can only be sent after response has ended.

HOL blocking.  Use many parallel connections?  However, this leads to inefficient networking behaviors.

HTTP2 solves HOL blocking by multiplexing multiple streams on a single connection.  Requests sent earlier, interleaved.

HTTP3 makes connection setup faster.  

IN HTTP3 streams are independent.  So when packet loss occurs it only affects one stream.

Enabled by QUIC.  New transport protocol from IETF.  Like TCP but E2E encryption.

* TLS 1.3 security
* Better performance

# QUIC
* TLS 1.3
* Faster connection setup
* 1 RTT
* Multiplexed streams don't suffer from HOL blocking
* Better loss recovery
* Connection migration

[[Reduce network delays for your app]]

# Using HTTP/3
If using URLSession, iOS 15 etc. ship this enabled by default.
Enable on your server and you're good to go.
Supports HTTP/3 RFC and Draft 29.

Use instruments to find out if using.  Network template.

Need to configure instruments to show details of HTTP transactions.  Make sure menu on leftside shows "HTTP Transactions" (bottom panel heading)

HTTP Version shows the version.  URLSession will only use hTTP3 if advertised.  In the righthand panel, we see it advertised through "alt-svc" header.  We will remember this to use for future connections, "service discovery".

Record app again.  Now we see HTTP/3.  

Service discovery is transparent to your app.

* DNS record: HTTPS `IN HTTPS alphn="h3,h2"`
* HTTP alternative services
	* `Alt-Svc: h3=":443"; ma=2592000` (max age in seconds)

Since information is DNS, HTTP/3 can be established the first time you contact.

When you know your server supports HTTP3, use `assumesHTTP3Capable` API.  Does not guarantee HTTP/3.   Networks may block HTTP3 or your server may not actually support.

## Priorities
HTTP priorities allows clients to prioritize responses on a connection.  

Originally develoepd in HTTP/2
Priorities in HTTP3 are simpler: `u=3, i`
* `priority` represented as urgency (`u`)
* `prefersIncrementalDelivery` represented as incremental (`i`)

URLSession may infer incremental delivery for async methods

For app downloading content that cannot be processed until resource is downloaded, set property to false.

Can change proriities later, e.g. in response to content becomign visible
# Using QUIC
* HTTP/3 is built on QUIC
* When should I use quic?
	* Non-request/response pair
	* Benefits from multiplexed streams
	* Custom protocols

```swift
// Create a connection using QUIC
let connection = NWConnection(host: "example.com", port: 443, using: .quic(alpn: ["myproto"]))

// Set the state update handler to be notified when the connection is ready
connection.stateUpdateHandler = { newState in
    switch newState {
    case .ready:
        print("Connected using QUIC!")
    default:
        break
    }
}

// Start the connection with callback queue
connection.start(queue: queue)
```

Now use send/receive as normal.

## Multiplexing protocols
* Send and receive on QUIC streams with `NWConnection`
* Use `NWMultiplexGroup` to refer to the underlying transport shared by the group of streams

Reason about the state of underlying QUIC tunnel.
Can create outgoing streams from a specific ?
And receive incoming streams

```swift
// Establish a tunnel with NWMultiplexGroup

// Create a group
let descriptor = NWMultiplexGroup(to: .hostPort(host: "example.com", port: 443))
let group = NWConnectionGroup(with: descriptor, using: .quic(alpn: ["myproto"]))

// Set the state update handler to be notified when the group is ready
group.stateUpdateHandler = { newState in
    switch newState {
    case .ready:
        print("Connected using QUIC!")
    default:
        break
    }
}

// Start the group with callback queue
group.start(queue: queue)
```

New outgoing stream
```swift
// Manage streams with NWConnectionGroup

// Create a new outgoing stream
let connection = NWConnection(from: group)

// Receive new incoming streams initiated by the remote endpoint
group.newConnectionHandler = { newConnection in

    // Set state update handler on incoming stream
    newConnection.stateUpdateHandler = { newState in
        // Handle stream states
    }

    // Start the incoming stream
    newConnection.start(queue: queue)

}
```

* use NWProtocolQUIC.Options for parameters
* `initialMaxStreamsBidirectional`
* properties of individual streams

If you're using `NWListener`, can also let you receive QUIC tunnels.  

NWProtocolMetadata to access information about streams

```swift
// Access QUIC metadata to learn about and modify streams

// Find the stream ID of a particular QUIC stream
if let metadata = connection.metadata(definition: NWProtocolQUIC.definition)
                             as? NWProtocolQUIC.Metadata {
    print("QUIC Stream ID is \(metadata.streamIdentifier)")

    // Some time later...

    // Set the application error, if appropriate, before cancelling the stream
    metadata.applicationError = 0x100
    connection.cancel()

}
```

Use metadata to communicate errors from remote endpoint.

* Create new QUIC tunnels with NWMultiplexGroup
* Extract a stream as an NWConnection from group
* Receive incoming QUCI tunnels and streams
* NWProtocolQUIC.Options
* NWProtocolQUIC.Metadata

## Debugging
Use environment variable to output `qlog` files
`QUIC_LOG_DIRECTORY=~/tmp`

New standardized logging format.  Richer information than traditional packet capture.  "Download Container..." for analysis.

Number of different opensource visualizations.

# Next steps
* Enable HTTP/3 on your server
* Upgrade your custom protocols to QUIC


