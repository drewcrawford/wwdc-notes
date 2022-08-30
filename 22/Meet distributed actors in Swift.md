#swift 

# Before we start
[[Protect mutable state with Swift actors]]

# Tic tac fish
# Sea of concurrency
Once an actor-based program compiles it is free from low-level data races.
Apply this to distributed.  Network or process boundary.
We can synchronize things easily.  Within the same concept of meessage passing, there are a few more restrictions.

Distributed actors can establish a channel between two processes.  From a programming model, not much has changed.  Actors still isolate their state, etc.  

This ability to be potentially remote, is called *location transparency*.  Regardless of where a distributed actor is located, we can interact the same way.  We can transparently move actors without changing implementation.

# Play offline
It needs to track how many moves it has made, and what teams it is playing for.  Use the number of moves made to select the character id.  

```swift
public actor OfflinePlayer: Identifiable {
    nonisolated public let id: ActorIdentity = .random

    let team: CharacterTeam
    let model: GameViewModel
    var movesMade: Int = 0

    public init(team: CharacterTeam, model: GameViewModel) {
        self.team = team
        self.model = model
    }

    public func makeMove(at position: Int) async throws -> GameMove {
        let move = GameMove(
            playerID: id,
            position: position,
            team: team,
            teamCharacterID: team.characterID(for: movesMade))
        await model.userMadeMove(move: move)

        movesMade += 1 
        return move
    }

    public func opponentMoved(_ move: GameMove) async throws {
        do {
            try await model.markOpponentMove(move)
        } catch {
            log("player", "Opponent made illegal move! \(move)")
        }
    }

}
```

Bot doesn't have to update the view model.  Maybe we'll start a conversion to distributed actors.
```swift
public actor BotPlayer: Identifiable {
    nonisolated public let id: ActorIdentity = .random
    
    var ai: RandomPlayerBotAI
    var gameState: GameState
    
    public init(team: CharacterTeam) {
        self.gameState = .init()
        self.ai = RandomPlayerBotAI(playerID: self.id, team: team)
    }
    
    public func makeMove() throws -> GameMove {
        return try ai.decideNextMove(given: &gameState)
    }
    
    public func opponentMoved(_ move: GameMove) async throws {
        try gameState.mark(move)
    }
}
```

# Play with a remote AI
## distributed, but still local, bot player

I need to import `Distributed`.  Contains various types.
```swift
import Distributed 

public distributed actor BotPlayer: Identifiable {
    typealias ActorSystem = LocalTestingDistributedActorSystem
  
    var ai: RandomPlayerBotAI
    var gameState: GameState
    
    public init(team: CharacterTeam, actorSystem: ActorSystem) {
        self.actorSystem = actorSystem // first, initialize the implicitly synthesized actor system property
        self.gameState = .init()
        self.ai = RandomPlayerBotAI(playerID: self.id, team: team) // use the synthesized `id` property
    }
    
    public distributed func makeMove() throws -> GameMove {
        return try ai.decideNextMove(given: &gameState)
    }
    
    public distributed func opponentMoved(_ move: GameMove) async throws {
        try gameState.mark(move)
    }
}
```

* does not declare an ActorSystem.
	* distributed actors belong to some system, which handles networking etc.
	* Since we're not running on a remote host, can use `LocalTesting`.
	* Can declare a module-wide typealias as well.  We'll use the local typealias I guess.

`nonisolated` cannot be applied to distributed actor stored properties.
Note that ids are owned by the actor system, we can't declare our own.  So I remove the manually-declared id property.

`.actorSystem` => compiler-synthesized properties which we need to declare by type and intiialize in our initializer.  We can pass it in via `init`., ex different for tests vs production etc.

Not every method on a distributed actor is for calling remotely.  Helper functions, etc.  Swift asks you to be explicit about the distributed API surface.  

## Server-side bot player
*Local and remote are a matter of perspective*.  Every remote reference has a local instance somewhwere.

1.  Local => pass into `init` etc.
2. Remote => resolve an actor ID.

```swift
let sampleSystem: SampleWebSocketActorSystem

let opponentID: BotPlayer.ID = .randomID(opponentFor: self.id)
let bot = try BotPlayer.resolve(id: opponentID, using: sampleSystem) // resolve potentially remote bot player
```

Actor systems should not perform "actually remoet" lookups when resolving these identifiers.  The resolve method is not async, and therefore should return quickly.

If an identity looks valid and seems to be pointing, just assume the actor exists etc.
At the time of resolving an id, the actual instance may not even exist!  

server implementation:

```swift
import Distributed
import TicTacFishShared

/// Stand alone server-side swift application, running our SampleWebSocketActorSystem in server mode.
@main
struct Boot {
    
    static func main() {
        let system = try! SampleWebSocketActorSystem(mode: .serverOnly(host: "localhost", port: 8888))
        
        system.registerOnDemandResolveHandler { id in
            // We create new BotPlayers "ad-hoc" as they are requested for.
            // Subsequent resolves are able to resolve the same instance.
            if system.isBotID(id) {
                return system.makeActorWithID(id) {
                    OnlineBotPlayer(team: .rodents, actorSystem: system)
                }
            }
            
            return nil // unable to create-on-demand for given id
        }
        
        print("========================================================")
        print("=== TicTacFish Server Running on: ws://\(system.host):\(system.port) ==")
        print("========================================================")
        
        try await server.terminated // waits effectively forever (until we shut down the system)
    }
}
```

create bots on demand, etc.
# Play with friends
## peer-to-peer: local networking
while we don't explain the networking framework,s ee [[Advances in networking, Pt 2 - 19]]

local network can expose privacy-sensitive information.

Now we have to discover the specific actor on the other device.  Generally solved with server discovery.  In distributed actors we have a common style.

Receptionist pattern.
* actors check in
* every system has its own receptionist
* may rely on existing service discovery and layer a type-safe API
* or a gossip-based mechanism


## online: clustered server-side lobby system
previously we directly created/resolved an opponent.  Now we have to wait for an opponent to appear on the network.  Let's introduce a match making view.

ask receptionist for opposing team's tag.

Let's call it`LocalNetowrkPlayer`

```swift
/// As we are playing for our `model.team` team, we try to find a player of the opposing team
let opponentTeam = model.team == .fish ? CharacterTeam.rodents : CharacterTeam.fish

/// The local network actor system provides a receptionist implementation that provides us an async sequence
/// of discovered actors (past and new)
let listing = await localNetworkSystem.receptionist.listing(of: OpponentPlayer.self, tag: opponentTeam.tag)
for try await opponent in listing where opponent.id != self.player.id {
    log("matchmaking", "Found opponent: \(opponent)")
    model.foundOpponent(opponent, myself: self.player, informOpponent: true)
    // inside foundOpponent:
    // if informOpponent {
    //     Task {
    //         try await opponent.startGameWith(opponent: myself, startTurn: false)
    //     }
    // }

    return // make sure to return here, we only need to discover a single opponent
}
```

```swift
public distributed actor LocalNetworkPlayer: GamePlayer {
    public typealias ActorSystem = SampleLocalNetworkActorSystem

    let team: CharacterTeam
    let model: GameViewModel

    var movesMade: Int = 0

    public init(team: CharacterTeam, model: GameViewModel, actorSystem: ActorSystem) {
        self.team = team
        self.model = model
        self.actorSystem = actorSystem
    }

    public distributed func makeMove() async -> GameMove {
        let field = await model.humanSelectedField()

        movesMade += 1
        let move = GameMove(
            playerID: self.id,
            position: field,
            team: team,
            teamCharacterID: movesMade % 2)

        return move
    }

    public distributed func makeMove(at position: Int) async -> GameMove {
        let move = GameMove(
            playerID: id,
            position: position,
            team: team,
            teamCharacterID: movesMade % 2)

        log("player", "Player makes move: \(move)")
        _ = await model.userMadeMove(move: move)

        movesMade += 1
        return move
    }

    public distributed func opponentMoved(_ move: GameMove) async throws {
        do {
            try await model.markOpponentMove(move)
        } catch {
            log("player", "Opponent made illegal move! \(move)")
        }
    }

    public distributed func startGameWith(opponent: OpponentPlayer, startTurn: Bool) async {
        log("local-network-player", "Start game with \(opponent.id), startTurn:\(startTurn)")
        await model.foundOpponent(opponent, myself: self, informOpponent: false)

        await model.waitForOpponentMove(shouldWaitForOpponentMove(myselfID: self.id, opponentID: opponent.id))
    }
}
```

## Online: clustered server-side lobby system
**Swift Distributed Actors Cluster Library**.  Implemented with swiftNEO, server-side data clustering.  Cluster-wide receptionist.  Take a look as it's advanced reference implementation.

# Wrap up
* Distributed actors
* Location transparency
* Distributed actor systems

* Swift
	* LocalTEstingDistributedActorSystem
	* Sample WebSocket actor system (swift nio, swift nio transport services /network framework)
	* Local networking actor system
		* network framework, bonjour
	* Swift Distributed Actors
		* Swift NIO, open source, fully featured server-side
# Next steps
* sample code app
* swift evolution
	* SE-0336 distributed actor isolation
	* SE-0344 Distributed Actor runtime
* Swift forums
	* Distributed actors category

[[Protect mutable state with Swift actors]]
[[Advances in networking, Pt 2 - 19]]


* https://developer.apple.com/forums/tags/wwdc2022-110356
* https://developer.apple.com/forums/create/question?&tag1=235&tag2=495030
* https://developer.apple.com/documentation/swift/tictacfish_implementing_a_game_using_distributed_actors
* https://forums.swift.org/c/server/distributed-actors/79
* https://github.com/apple/swift-evolution/blob/main/proposals/0344-distributed-actor-runtime.md
* https://github.com/apple/swift-evolution/blob/main/proposals/0336-distributed-actor-isolation.md
* https://github.com/apple/swift-distributed-actors/
