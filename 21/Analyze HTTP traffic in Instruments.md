#instruments #networking

# It just works
* On all apple devices
* Supports HTTP/3, VPN
* Integration on the framework level
	* Process attribution
	* Cached on-disk requests
	* Netowrking errors
	* Explores API concepts: URLSession and URLSessionTask

# HTTP instrument UI overview
Track hierarchy
* HTTP traffic instrument
* Number of active tasks

Process
* Debuggable processes and background tasks daemon

Session
* One track per URLSession object
* Individual task intervals
* Configurable session name

Can name with `sessionDescription`.

Domain
* Only tasks that requested this domain
* More details
	* Individual transactions
	* Transaction states

## Tasks
structure

1.  Task
	1.  Transactions
		1.  Request
		2.  Response

Task level is a `dataTask`, etc.  When you call `.resume` the interval starts.  And it ends at completion block.

Each task can be given a semantic name.  Labels the interval in instruments.

Also show `taskIdentifier`.  If your task finishes with an error, its description will be present on the label.

Task can be made up of multiple transactions.

Preferred domain redirect.  This creates 2 transactions.

## Transactions
Request+Response pair handled by URL loading system
Contains all HTTP layer information
* URL, HTTP version, connection, cache info
* Request + response header
* Request + response body
* And more!


`label` summarizes the transaction.  Mainly the request and response. 

Track hierarchy gives you the domain.  Path and query on the label.

version, method, authorization/cookie.  
Respone: status code, cookie, content-type

Additional info about flow of a transaction is captured by transaction state

1.  URL loading system creates transaction
2.  Cache lookup
3.  Blocked.  Waiting for connection.  Connection setup/busy
4.  Sending request -> first to last byte
5.  Waiting for response
6.  Receiving response -> first to last byte
7.  Transaction completes

In practice, cache lookup and seding request are usually quite short.

# Solving performance and correctness issues
Product=>Profile.
"Network" template.

General network connections.  New HTTP tracing functionality.  

2 tracks.  HTTP traffic and Network Connections.

Alert to warn you about sensitive information.  

See time by selecting region.

Increased block states.  congestion issue?

Switch tracks to be http transactions by connection.  In sidebar, under domain name, click the downwards arrow.  "HTTP Transactions by Connection"

This view will group transactions by the connection they use.  Overall there were 6 connections available.  

Each transaction is taking longer.  Staircase pattern.  Each one is blocked until the prior transaction is finished.  This pattern repeats.

**Head of line blocking**.  They spend most of their time blocked or waiting for the response.  

Main limitations of HTTP1 and the main improvements in HTTP2.  

## Use modern HTTP versions
* HTTP2 can multiplex multiple transactions on the same connection

[[Accelerate networking with HTTP3 and QUIC]]

We notice there's only 1 connection.  We no longer need multiple connections.  Only need to pay the setup cost once.

We spend no time on the blocked state.  Time is so small it isn't visible.

Responses are making progress simultaneously.  All requests in under 3s.

## demo
Why do I have to log in again?  

On the left, there's a task corresponding to the favorite button.  

Far right: task corresponding to second time.

First one contains two transactions.  First received a 401 status code (not logged in).  Drawn in orange to indicate not a success for HTTP.

Large empty area.  Then there's another put request for the retry.  201 indicates it succeeded.

Second attempt.  Task is displayed in grey since the task was cancelled due to dismissing the login screen.  

I would have expected this task to set the proper cookie.  Small "C" next to request.  No cookie was sent.  But we got it from server.

Transaction List at the bottom.  Detail view in inspector.  Shows all request/response headers.  But we see expiry date in the past.  So the server did send a cookie, but it's in the past.

Notice this transaction is not executed on a connection, but on local cache.  This explains why there's no way for a response state to not wait.

`reloadRevalidatingCacheData` => ignore the local cache.  If so, user will send 304 to let us use the cache.


# Auditing your application behavior
Find out who's sending SDK analytics.

`xctrace export --input foo.trace --har`

inspect recorded information in any tool that supports "har", e.g. without instruments.

JSON-based format.  Opened in the text editor, etc.

# Next steps
* Target your app today to detect problems
* Name your URLSession and URLSessionTask for easier debugging
* Adopt latest networking protocols
* Audit your application requests to check if you can send less information

