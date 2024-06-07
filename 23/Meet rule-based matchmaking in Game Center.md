Learn how to incorporate the new rule-based matchmaking feature into your real-time multiplayer games. Discover how you can provide customized and flexible matchmaking to improve the quality of player matches and create a more fun and engaging experience for all players.

# Matchmaking overview
* match players in unique game modes for a personalized gaming experience
* Ensure fair and balanced matchups based on skill level or preferences
* Avoid mismatched games that can lead to frustration or boredom
* Match players to balanced teams
* Private matches for friends
* relax constraints to balance wait time against match quality

with classic match making you can do game modes or private matches.  Now you can do fair skill, balanced teams, relax constraints, custom rules, update rules

Use ASC API to configure rules to best suit my game.  On game side, GameKit SDK provides apis for me to specify the amtch request queue, along with custom properties for a player to be used by my rules.

GC service periodically runs an algorithm, to find requests.

* matchmaking framework
* configure queues and rule sets
* express matchmaking constraints using a powerful rules language
* GameKit APIs provide custom player properties

Developer tooling
* test rules with sample requests
* Metrics to understand real player experience



# How to set up matchmaking

1.  API keys
2. upload rules
3. test
4. create queue
5. add ...
6. ?

## ASC API
Discover REST API endpoints to configure matchmaking rules
obtain API key from ASC account
Learn how to generate a JWT for authorization, see docs.

Play others with a similar skill level
strict enforcement could mean waiting
Better experience to balance wait time against match quality

## create a ruleset
`gameCenterMatchmakingRuleSets`.  

Require compatible app versions.
Latency.
Skill difference.

rulesets, queues associated with ASC provider organization.  We recommend using game bundle ID reverse domain prefix.

expression grammar, open-source query language called JMESPath.

`agedValues`.  Initial skill range of 20.  First array is breakpoints in the fn, second array is seconds where the breakpoints happen.

## testing rules
Python script: `testrules.py`.  

see docs.

## create queue

gameCenterMatchmakingQueues endpoint to create a queue for app requests.

## add to game

use existing gk framework classes
* GKMatchREquest
* GKMatch
Use new object properties
* queueName
* properties

## observe

* request counts by outcome
	* matched, canceled, expired
* time taken to reach outcome
* queue length
* rule evaluation errors and outcomes

see docs

# Wrap up
rule based matchmaking available now
see the developer documentation for more details
share feedback with FA


# Resources
* https://developer.apple.com/documentation/appstoreconnectapi
* https://developer.apple.com/documentation/appstoreserverapi/generating_json_web_tokens_for_api_requests
* https://developer.apple.com/documentation/gamekit/matchmaking_rules
* https://developer.apple.com/documentation/appstoreconnectapi/game_center/metrics
* https://developer.apple.com/documentation/appstoreconnectapi/game_center/metrics
