Meet the RealityKit debugger and discover how this new tool lets you inspect the entity hierarchy of spatial apps, debug rogue transformations, find missing entities, and detect which parts of your code are causing problems for your systems.

### clubview - 2:45
```swift

/*
Abstract:
The full club patch. SwiftUI view, state, extensions and helpers.
*/

import SwiftUI
import RealityKit
import OSLog
import BOTanistAssets
import Combine
import Charts

struct ClubView: View {
	@State var state = ClubViewState()
	
	var body: some View {
		ZStack {
			RealityView { content in
				state.loadEnvironment()
				
				state.rootEntity.scale = SIMD3<Float>(repeating: 0.5)
				
				content.add(state.rootEntity)
			} update: { updateContent in
				if !state.doorSupervisor.doorsOpen {
					state.transformIntoClub(content: updateContent)
				}
			}
		}
	}
}

@Observable
@MainActor
final public class ClubViewState: Sendable {
	let rootEntity = Entity()
	
	private var loadedEnvironmentRoot: Entity?
	private var robotRevolutionController: Entity?
	private var host: Entity?
	
	private(set) var doorSupervisor: DoorSupervisor {
		get {
			rootEntity.components[DoorSupervisor.self]!
		} set {
			rootEntity.components[DoorSupervisor.self] = newValue
		}
	}
	
	init() {
		RevolvingSystem.registerSystem()
		HoverSystem.registerSystem()
		TeleportationSystem.registerSystem()
		DanceMotivationSystem.registerSystem()
		
		rootEntity.name = "The B0T Club"
		rootEntity.components[DoorSupervisor.self] = DoorSupervisor(capacity: 9)
	}
	
	/// Load the existing garden assets
	func loadEnvironment() {
		guard loadedEnvironmentRoot == nil else {
			return
		}
		
		if let environment = try? Entity.load(named: "scenes/volume", in: BOTanistAssetsBundle) {
			environment.name = "Environment"
			self.loadedEnvironmentRoot = environment
			
			rootEntity.addChild(environment)
		}
	}
	
	/// Renovate the loaded environment to build our club
	func transformIntoClub(content: RealityViewContent) {
		guard !doorSupervisor.doorsOpen else {
			return
		}
		
		// Build a teleportation center and use it to spawn robots
		addTeleportationCenterToTheClub()
		
		// Haphazardly clean up the space by hiding anything un-club-like
		hideStuffInTheEnvironment()
		
		// Polish that floor and add some spin
		addRevolvingDanceFloorToTheClub()
		
		// Keep the robots moving in an orderly fashion
		addRobotRevolutionControllerToTheClub()
		
		// Install some attractors to entice robots to the dance floor
		addDanceFloorAttractors()
		
		// Set the mood
		addSpotlightsToTheClub()
		
		// Stock up on oil to keep the moves smooth
		addCounterToTheClub()
		
		// And add a huge Disco Ball, because...
		addDiscoBallToTheClub()
		
		// Let the party begin
		openDoors()
	}
	
	/// Construct a Teleportation Center and add it to the Club's root entity
	private func addTeleportationCenterToTheClub() {
		let teleportationCenter = Entity()
		teleportationCenter.name = "Teleportation Center"
		rootEntity.addChild(teleportationCenter)
		
		// Liven up the planters to look more like teleporters
		let positions: [SIMD3<Float>] = [[0.128, 0, 0.14], [-0.255, 0, 0.23], [0.05, 0, -0.17]]
		let colors: [(UIColor, UIColor)] = [(.green, .yellow), (.magenta, .purple), (.cyan, .blue)]
		for index in 0...2 {
			if let teleporter = rejigPlanter(identifier: String(index + 1), position: positions[index], colors: colors[index]) {
				teleportationCenter.addChild(teleporter)
			}
		}
		
		// Create a Control Center and provide a closure to handle robot spawning
		let teleportationControlCenter = ControlCenterComponent(
			initialValue: 10,
			interval: 5,
			rootEntity: rootEntity) { teleporter in
				self.spawnRobot(from: teleporter)
				self.countVisitor()
				
				// Have the host say hello
				if let hostCharacter = self.host?.components[AutomatonControl.self]?.character {
					hostCharacter.transitionToAndPlayAnimation(.idle)
					hostCharacter.transitionToAndPlayAnimation(.wave)
				}
		}
		
		// Assign the new control center component to the teleportation center entity
		teleportationCenter.components[ControlCenterComponent.self] = teleportationControlCenter
	}
	
	/// Transforms the visuals of the planters to look more teleporter-y
	private func rejigPlanter(identifier: String, position: SIMD3<Float>, colors: (UIColor, UIColor)) -> Entity? {
		if let rim = rootEntity.findEntity(named: "heroPlanter_rim_\(identifier)"),
		   let dirt = rootEntity.findEntity(named: "dirt_hero_\(identifier)"),
		   let rimModelComponent = rim.components[ModelComponent.self],
		   var dirtModelComponent = dirt.components[ModelComponent.self] {
			// Apply the luminous material from the rims to the dirt (trust me it will look cool).
			dirtModelComponent.materials = rimModelComponent.materials
			dirt.components[OpacityComponent.self] = OpacityComponent(opacity: 0.7)
			dirt.components[ModelComponent.self] = dirtModelComponent
		}
		
		// Make a teleporter container entity
		let teleporter = Entity()
		teleporter.name = "Teleporter-T\(identifier)"
		teleporter.position = position
		teleporter.components[TeleporterComponent.self] = TeleporterComponent()
		
		// Add a particle emitter
		let radius: Float = 0.035
		var particleEmitter = ParticleEmitterComponent.Presets.teleporter
		particleEmitter.emitterShapeSize = .init(repeating: radius)
		particleEmitter.mainEmitter.color = .constant(.random(a: colors.0, b: colors.1))
		
		let particleEntity = Entity()
		particleEntity.orientation = .init(angle: -.pi / 2, axis: [1, 0, 0])
		particleEntity.components[ParticleEmitterComponent.self] = particleEmitter
		particleEntity.name = "Photons"
		particleEntity.scale = .init(repeating: 1)
		teleporter.addChild(particleEntity)
		
#if DEBUG
		// Add a debug marker in case we want to visually inspect this in the RealityKit Debugger
		teleporter.addDebugMarker(radius: radius, color: colors.0)
#endif
		
		return teleporter
	}
	
	/// adds a random robot to the club root, positioned at the provided point
	private func spawnRobot(from spawnPoint: Entity) {
		guard let robotCharacter = randomRobot() else {
			logger.error("Robot creation malfunction 🤖💥")
			return
		}
		
		let guest = Entity()
		
		guest.addChild(robotCharacter.characterParent)
		guest.position = spawnPoint.position(relativeTo: rootEntity)
		guest.components[Newcomer.self] = Newcomer()
		guest.components[AutomatonControl.self] = AutomatonControl(character: robotCharacter)
		
		rootEntity.addChild(guest)
		
		// Play a little flashy burst on the particle emitter
		if let particles = spawnPoint.findEntity(named: "Photons") {
			var component = particles.components[ParticleEmitterComponent.self]
			component?.burst()
			particles.components[ParticleEmitterComponent.self] = component
		}
	}
	
	/// misuses AppState as a robot factory - don't try this at home, or do, but don't ship it!
	private func randomRobot() -> RobotCharacter? {
		let robotMaker = AppState()
		
		// Use offsets from the loaded animation rig, with some random parts
		guard let skeleton = robotMaker.robotData.meshes[.body]?.findEntity(named: "rig_grp") as? ModelEntity else {
			logger.error("Failed to find a robot animation rig... all dancing in cancelled ❌🕺")
			return nil
		}
		
		robotMaker.randomizeSelectedRobot()
		
		guard let head = robotMaker.robotData.meshes[.head]?.clone(recursive: true),
			  let body = robotMaker.robotData.meshes[.body]?.clone(recursive: true),
			  let backpack = robotMaker.robotData.meshes[.backpack]?.clone(recursive: true) else {
			fatalError()
		}
		
		let robotCharacter = RobotCharacter(
			head: head,
			body: body,
			backpack: backpack,
			appState: robotMaker,
			headOffset: skeleton.pins["head"]?.position,
			backpackOffset: skeleton.pins["backpack"]?.position
		)
		
		// Pick a random robot name from the sequence
		robotCharacter.characterParent.name = RobotNames.next
		
		// Remove the character controller and animation state, as we'll manually control these
		robotCharacter.characterParent.components[CharacterControllerComponent.self] = nil
		AnimationState.handlers.removeAll()
		
		// The robots are here to chill, so actually, let's put their backpacks in the cloakroom
		backpack.removeFromParent()
		
		// Say Hi
		robotCharacter.transitionToAndPlayAnimation(.wave)
		
		return robotCharacter
	}
	
	/// Update capacity when we have a visitor
	private func countVisitor() {
		var management = self.doorSupervisor
		management.visitorCount += 1
		self.doorSupervisor = management
	}
	
	/// Find and hide a bunch of stuff in the loaded environment
	private func hideStuffInTheEnvironment() {
		// We used the RealityKit Debugger to identify the names of things we want to hide in the club
		["setDressing", "MovementBoundaries", "planter_side", "planter_Hero", "planter_Hero_1", "planter_Hero_2", "PlantLightGroup",
		 "PlantLightGroup_1", "PlantLightGroup_2", "SidePlanterLights", "pipe_2", "pipe_3", "dirt_coffeeBerry_1", "dirt_coffeeBerry_2",
		 "dirt_coffeeBerry_3", "dirt_side"].forEach { name in
			if let entity = rootEntity.findEntity(named: name) {
				entity.removeFromParent()
			}
		}
	}
	
	/// Repurpose some existing bits in the environment to create a makeshift revolving dance floor - if it looks like dirt, that's because it is
	private func addRevolvingDanceFloorToTheClub() {
		guard let dirtFloor = loadedEnvironmentRoot?.findEntity(named: "dirt_end") else {
			return
		}
		
		// Add a revolving container entity
		let revolvingDanceFloor = Entity()
		revolvingDanceFloor.name = "Revolving Dance Floor"
		revolvingDanceFloor.scale = [1, 1, 1]
		revolvingDanceFloor.position = [0, 0.181, 0]
		revolvingDanceFloor.components[RevolvingComponent.self] = RevolvingComponent(relativeTo: rootEntity)
		
		// Polish up the dirt floor
		let geometry = dirtFloor.clone(recursive: false)
		geometry.name = "Dirt Floor"
		geometry.transform = .identity
		geometry.position = [0, 0, 0]
		geometry.scale = dirtFloor.scale(relativeTo: rootEntity)
		
		let polish = geometry.clone(recursive: false)
		polish.name = "Polish Layer"
		polish.position = [0, 0.0004, 0]
		
		if var modelComponent = geometry.components[ModelComponent.self] {
			var polishedFloorMaterial = PhysicallyBasedMaterial()
			
			polishedFloorMaterial.baseColor = .init(tint: .gray)
			polishedFloorMaterial.roughness = .init(floatLiteral: 0.2)
			polishedFloorMaterial.metallic = .init(floatLiteral: 0.8)
			polishedFloorMaterial.blending = .transparent(opacity: .init(floatLiteral: 0.5))
			polishedFloorMaterial.clearcoat = .init(floatLiteral: 0.4)
			
			modelComponent.materials = [polishedFloorMaterial]
			
			polish.components[ModelComponent.self] = modelComponent
		}
		
		// Add it to the revolving container
		revolvingDanceFloor.addChild(geometry)
		revolvingDanceFloor.addChild(polish)
		
		rootEntity.addChild(revolvingDanceFloor)
	}
	
	/// Creates a revolving container entity to keep robots moving in sync with the dance floor
	private func addRobotRevolutionControllerToTheClub() {
		let robotRevolutionController = Entity()
		robotRevolutionController.name = "Robot Revolution Controller"
		robotRevolutionController.components[RevolvingComponent.self] = RevolvingComponent(relativeTo: rootEntity)
		
		rootEntity.addChild(robotRevolutionController)
		
		self.robotRevolutionController = robotRevolutionController
	}
	
	/// Add invisible attractors to the dance floor to position and control robots
	private func addDanceFloorAttractors() {
		guard let robotRevolutionController else {
			logger.error("The Robot Revolution Controller is missing 😱")
			return
		}
		
		// Add a few dance spots on the outside of the club that we know don't obstruct the furniture
		let staticAttractors = Entity()
		staticAttractors.name = "Static Attractors"
		
		let placementRadius: Float = 0.25
		let outerRadius = placementRadius * 0.8
		addDanceFloorAttractor(to: staticAttractors, angle: Angle2D(degrees: 10), placementRadius: outerRadius, name: "Static-A1", variation: 0)
		addDanceFloorAttractor(to: staticAttractors, angle: Angle2D(degrees: 90), placementRadius: outerRadius, name: "Static-A2", variation: 0)
		addDanceFloorAttractor(to: staticAttractors, angle: Angle2D(degrees: 130), placementRadius: outerRadius, name: "Static-A3", variation: 0)
		addDanceFloorAttractor(to: staticAttractors, angle: Angle2D(degrees: 240), placementRadius: outerRadius, name: "Static-A4", variation: 0)
		addDanceFloorAttractor(to: staticAttractors, angle: Angle2D(degrees: 325), placementRadius: outerRadius, name: "Static-A5", variation: 0)
		
		rootEntity.addChild(staticAttractors)
		
		// The remaining center attractors are on the revolving dance floor and can be more randomly positioned
		let innerRingCapacity = doorSupervisor.capacity - 5
		
		let revolvingAttractors = Entity()
		revolvingAttractors.name = "Revolving Attractors"
		
		addDanceFloorAttractors(to: revolvingAttractors, count: innerRingCapacity, placementRadius: placementRadius * 0.3, namePrefix: "Revolving")
		
		robotRevolutionController.addChild(revolvingAttractors)
		
#if DEBUG
		// Add some debug visualizations
		let debugRoot = Entity()
		debugRoot.name = "[Debug] Dance System"
		debugRoot.isEnabled = false
		debugRoot.components[DanceSystemDebugComponent.self] = DanceSystemDebugComponent()
		
		rootEntity.addChild(debugRoot)
		
		let allAttractors = Array(staticAttractors.children) + Array(revolvingAttractors.children)
		
		// Create a new visualization for each attractor
		allAttractors.forEach { attractor in
			if let visualization = Entity.makeDebugMarker(height: 0.08, radius: 0.03, enabled: true) {
				guard let attractorComponent = attractor.components[AttractorComponent.self] else {
					return
				}
				
				let debugComponent = AttractorDebugComponent(state: attractorComponent.state, attractor: attractor)
				
				visualization.position = [0, 0.04, 0]
				visualization.components[AttractorDebugComponent.self] = debugComponent
				debugRoot.addChild(visualization)
			}
		}
#endif
	}
	
	/// Add multiple dance floor attractors along the circumference of a circle with the specified placementRadius
	private func addDanceFloorAttractors(to danceFloor: Entity, count: Int, placementRadius: Float, namePrefix: String, variation: Float = 0.005) {
		let angleIncrements = 360 / count
		
		for offset in 0..<count {
			let angle = Angle2D(degrees: Double(angleIncrements * offset))
			let name = "\(namePrefix)-A\(offset + 1)"
			addDanceFloorAttractor(to: danceFloor, angle: angle, placementRadius: placementRadius, name: name, variation: variation)
		}
	}
	
	/// Adds a single dance floor attractor at a point on the circumference of a circle with the specified placementRadius
	private func addDanceFloorAttractor(to danceFloor: Entity, angle: Angle2D, placementRadius: Float, name: String, variation: Float = 0.005) {
		let attractor = Entity()
		attractor.name = name
		attractor.components[AttractorComponent.self] = AttractorComponent(club: rootEntity)
		attractor.position = pointOnCircumference(angle: angle, radius: placementRadius, variation: variation)
		danceFloor.addChild(attractor)
	}
	
	/// Adds some revolving spot lights to the club
	private func addSpotlightsToTheClub() {
		let placementRadius: Float = 0.5
		let lightsWrapper = Entity()
		lightsWrapper.name = "Light Rig"
		
		let magentaLight = SpotLight()
		magentaLight.light.color = .magenta
		magentaLight.light.intensity = 500
		var lightPosition = pointOnCircumference(angle: Angle2D(degrees: 0), radius: placementRadius, y: 0.5)
		magentaLight.look(at: .zero, from: lightPosition, relativeTo: rootEntity)
		lightsWrapper.addChild(magentaLight)
		
		let greenLight = magentaLight.clone(recursive: true)
		greenLight.light.color = .green
		lightPosition = pointOnCircumference(angle: Angle2D(degrees: 120), radius: placementRadius, y: 0.5)
		greenLight.look(at: .zero, from: lightPosition, relativeTo: rootEntity)
		lightsWrapper.addChild(greenLight)
		
		let cyanLight = magentaLight.clone(recursive: true)
		cyanLight.light.color = .cyan
		lightPosition = pointOnCircumference(angle: Angle2D(degrees: 240), radius: placementRadius, y: 0.5)
		cyanLight.look(at: .zero, from: lightPosition, relativeTo: rootEntity)
		lightsWrapper.addChild(cyanLight)
		
		lightsWrapper.components[RevolvingComponent.self] = RevolvingComponent(speed: -0.2, relativeTo: rootEntity)
		
		rootEntity.addChild(lightsWrapper)
	}
	
	/// Repurpose some planters to make a counter and stocks with a premium aged oil, and a friendly host
	private func addCounterToTheClub() {
		guard let planter = rootEntity.findEntity(named: "planter_big"),
			  let dirt = rootEntity.findEntity(named: "dirt_big") else {
			logger.error("Making the counter failed... too much dancing may now cause rust 🤖")
			return
		}
		
		// Group into a container entity
		let counter = Entity()
		counter.name = "Counter"
		counter.position = [0.333, 0.05, -0.09]
		rootEntity.addChild(counter)
		
		// Repurpose existing assets
		let counterGeometry = Entity()
		counterGeometry.name = "Counter Geometry"
		counterGeometry.addChild(planter, preservingWorldTransform: true)
		counterGeometry.addChild(dirt, preservingWorldTransform: true)
		counterGeometry.scale = [2, 6, 2]
		counterGeometry.position = [-0.3335, -0.15, 0.09]
		counter.addChild(counterGeometry)
		
		var counterTopMaterial = PhysicallyBasedMaterial()
		counterTopMaterial.baseColor = .init(tint: .white)
		counterTopMaterial.roughness = .init(floatLiteral: 0)
		counterTopMaterial.metallic = .init(floatLiteral: 1)
		
		dirt.components[ModelComponent.self]?.materials = [counterTopMaterial]
		dirt.position += [0, 0.001, 0]
		
		// Add a fancy hover rail
		if let rim = rootEntity.findEntity(named: "bottom_rim_1") {
			let hoverRailing = rim.clone(recursive: true)
			hoverRailing.name = "Hover Railing"
			hoverRailing.position = [0, 0.1, 0]
			hoverRailing.scale = rim.scale(relativeTo: rootEntity) * 0.5
			hoverRailing.components[HoverComponent.self] = HoverComponent(from: hoverRailing.position, to: hoverRailing.position + [0, -0.03, 0])
			counter.addChild(hoverRailing)
		}
		
		// Add some bottles to the counter
		let bottles = stockBottles(placementRadius: 0.045)
		counter.addChild(bottles)
		
		// Hide any out of stock items
		for bottle in bottles.children {
			bottle.isEnabled = bottle.components[OutOfStockComponent.self] == nil
		}
		
		// Add a friendly host
		addHostToTheCounter(counter)
	}
	
	/// Adds 9 green bottles of the finest aged oil to the counter (assuming we have them in stock)
	private func stockBottles(placementRadius: Float) -> Entity {
		let bottleRadius: Float = 0.003
		let bottleHeight: Float = 0.022
		let angleIncrement: Float = -12
		let outOfStockBrands: Set = [3]
		
		// Make a wrapper entity
		let bottleGroup = Entity()
		bottleGroup.name = "Bottle Group"
		bottleGroup.position = [0, 0.04, 0]
		bottleGroup.orientation = .init(angle: 180 * (.pi / 180), axis: [0, 1, 0])
		
		// Make a nice green material
		var bottleMaterial = PhysicallyBasedMaterial()
		bottleMaterial.baseColor = .init(tint: .green)
		bottleMaterial.blending = .transparent(opacity: .init(floatLiteral: 0.5))
		
		// A simple cylinder mesh
		let bottleMesh = MeshResource.generateCylinder(height: bottleHeight, radius: bottleRadius)
		
		// Error 1: Content occluded
		let bottle1 = Entity()
		bottle1.name = "BT1"
		bottle1.position = pointOnCircumference(angle: .zero, radius: placementRadius, y: -0.03)
		bottle1.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle1)
		
		// Error 2: Content clipped
		let bottle2 = Entity()
		bottle2.name = "BT2"
		bottle2.position = pointOnCircumference(angle: Angle2D(degrees: angleIncrement), radius: 1.6, y: bottleHeight / 2)
		bottle2.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle2)
		
		// Error 3: Content inside out
		let bottle3 = Entity()
		bottle3.name = "BT3"
		bottle3.position = pointOnCircumference(angle: Angle2D(degrees: 2 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle3.scale = .init(repeating: 650)
		bottle3.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle3)
		
		// Error 4: Content not enabled
		let bottle4 = Entity()
		bottle4.name = "BT4"
		bottle4.position = pointOnCircumference(angle: Angle2D(degrees: 3 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle4.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottle4.components[OutOfStockComponent.self] = OutOfStockComponent()
		bottleGroup.addChild(bottle4)
		
		// Error 5: Content not anchored
		let bottle5 = Entity()
		bottle5.name = "BT5"
		bottle5.position = pointOnCircumference(angle: Angle2D(degrees: 4 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle5.components[AnchoringComponent.self] = AnchoringComponent(.plane(.horizontal, classification: .table, minimumBounds: .zero))
		bottle5.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle5)
		
		// Error 6: Content missing a mesh
		let bottle6 = Entity()
		bottle6.name = "BT6"
		bottle6.position = pointOnCircumference(angle: Angle2D(degrees: 5 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle5.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle6)
		
		// Error 7: Content's material misconfigured
		let bottle7 = Entity()
		bottle7.name = "BT7"
		bottle7.position = pointOnCircumference(angle: Angle2D(degrees: 6 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		
		var simplifiedBottleMaterial = UnlitMaterial(color: .green.withAlphaComponent(0.5))
		simplifiedBottleMaterial.opacityThreshold = 1
		
		bottle7.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [simplifiedBottleMaterial])
		bottleGroup.addChild(bottle7)
		
		// Error 8: Content has a broken mesh
		let alternativeMesh = MeshResource.generateAbnormalCylinder(height: bottleHeight, radius: bottleRadius)
		let bottle8 = Entity()
		bottle8.name = "BT8"
		bottle8.position = pointOnCircumference(angle: Angle2D(degrees: 7 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle8.scale = [bottle8.scale.x, bottle8.scale.y, -bottle8.scale.z]
		bottleMaterial.opacityThreshold = 0
		bottle8.components[ModelComponent.self] = ModelComponent(mesh: alternativeMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle8)
		
		// Error 9: Content not added to the scene hierarchy
		let bottle9 = Entity()
		bottle9.name = "BT9"
		bottle9.position = pointOnCircumference(angle: Angle2D(degrees: 8 * angleIncrement), radius: placementRadius, y: bottleHeight / 2)
		bottle9.components[ModelComponent.self] = ModelComponent(mesh: bottleMesh, materials: [bottleMaterial])
		bottleGroup.addChild(bottle8)
		
		// FIXME: Bottles are missing from the counter
		
		return bottleGroup
	}
	
	/// Add a host robot to the counter
	private func addHostToTheCounter(_ counter: Entity) {
		// Make a clone of our hero BOTanist
		let robotMaker = AppState()
		
		guard let skeleton = robotMaker.robotData.meshes[.body]?.findEntity(named: "rig_grp") as? ModelEntity else {
			fatalError()
		}
		
		// But use the hover body to best complement the counter
		robotMaker.setMesh(part: .body, name: "body3")
		
		guard let head = robotMaker.robotData.meshes[.head]?.clone(recursive: true),
			  let body = robotMaker.robotData.meshes[.body]?.clone(recursive: true),
			  let backpack = robotMaker.robotData.meshes[.backpack]?.clone(recursive: true) else {
			fatalError()
		}
		
		let robotCharacter = RobotCharacter(
			head: head,
			body: body,
			backpack: backpack,
			appState: robotMaker,
			headOffset: skeleton.pins["head"]?.position,
			backpackOffset: skeleton.pins["backpack"]?.position
		)
		
		// Remove the character controller and animation state, as we'll manually control these
		AnimationState.handlers.removeAll()
		robotCharacter.characterParent.components[CharacterControllerComponent.self] = nil
		
		// Take off that heavy backpack
		backpack.removeFromParent()
		
		// Setup our host using the character and add it to the counter
		let host = Entity()
		host.name = "Host"
		host.orientation = .init(angle: 300 * (.pi / 180), axis: [0, 1, 0])
		host.position = [0, 0.005, 0]
		host.components[AutomatonControl.self] = AutomatonControl(character: robotCharacter)
		host.addChild(robotCharacter.characterParent)
		counter.addChild(host)
		
		// Have them say Hi
		robotCharacter.transitionToAndPlayAnimation(.wave)
		
		// Save a reference so they can wave later when other bots enter
		self.host = host
	}
	
	/// Generates a disco ball looking entity, makes it revolve and hover, and adds it to the club
	private func addDiscoBallToTheClub() {
		// Add the top level revolving, hovering disco ball entity
		let discoBall = Entity()
		discoBall.name = "Disco Ball"
		discoBall.position = [-0.305, 0.17, 0.02]
		discoBall.components[RevolvingComponent.self] = RevolvingComponent(speed: -0.02, relativeTo: rootEntity)
		discoBall.components[HoverComponent.self] = HoverComponent(from: discoBall.position, to: discoBall.position + [0, 0.02, 0])
		
		rootEntity.addChild(discoBall)
		
		// Add a support beam to hold the disco ball
		var supportMaterial = PhysicallyBasedMaterial()
		supportMaterial.baseColor = .init(tint: .lightGray)
		supportMaterial.roughness = .init(floatLiteral: 0.8)
		supportMaterial.metallic = .init(floatLiteral: 0.8)
		
		let support = ModelEntity(mesh: .generateCylinder(height: 0.01, radius: 0.01), materials: [supportMaterial])
		support.scale = [0.2, 1.8, 0.2]
		support.position = [0, 0.05, 0]
		support.name = "Support"
		
		discoBall.addChild(support)
		
		// Add the shiny ball that is the base of our disco ball
		var backgroundMaterial = PhysicallyBasedMaterial()
		backgroundMaterial.baseColor = .init(tint: .lightGray)
		backgroundMaterial.roughness = .init(floatLiteral: 0)
		backgroundMaterial.metallic = .init(floatLiteral: 1)
		
		let background = ModelEntity(mesh: .generateSphere(radius: 0.05), materials: [backgroundMaterial])
		background.name = "Background"
		
		// FIXME: Unintentionally inheriting an ancestor's transformation
		support.addChild(background)
		
		// Add some detailed lines on top of the background
		var lineMaterial = PhysicallyBasedMaterial()
		lineMaterial.baseColor = .init(tint: .lightGray)
		lineMaterial.sheen = .init(tint: .lightGray)
		lineMaterial.emissiveColor = .init(color: .lightGray)
		lineMaterial.emissiveIntensity = 1
		lineMaterial.triangleFillMode = .lines
		
		let ballOutline = ModelEntity(mesh: .generateSphere(radius: 0.0505), materials: [lineMaterial])
		ballOutline.name = "Outline"
		
		background.addChild(ballOutline)
	}
	
	/// Marks the club as ready
	private func openDoors() {
		var management = self.doorSupervisor
		management.doorsOpen = true
		self.doorSupervisor = management
	}
	
	/// finds a point along the edge of a circle on an XZ-plane, given a radius and y value. Optionally applies some variance.
	private func pointOnCircumference(angle: Angle2D, radius: Float, variation: Float = 0, y: Float = 0) -> SIMD3<Float> {
		.init(
			x: (Float(cos(angle)) * radius) + .random(in: -variation...variation),
			y: y,
			z: (Float(sin(angle)) * radius) + .random(in: -variation...variation)
		)
	}
}

// MARK: Club Management

/// Manages club capacity and ready state
struct DoorSupervisor: Component {
	let capacity: Int
	var doorsOpen = false
	var visitorCount = 0
	
	var hasCapacity: Bool {
		visitorCount < capacity
	}
}

/// Tag to indicate if a retail item is in stock
struct OutOfStockComponent: Component {}

// MARK: Revolution Control

/// Works with the RevolvingSystem to apply a continuous rotation to an entity
struct RevolvingComponent: Component {
	var speed: Float
	var angle: Float
	var axis: SIMD3<Float>
	var relativeTo: Entity?
	
	init(speed: Float = 0.05, initialAngle: Float = 0, axis: SIMD3<Float> = [0, 1, 0], relativeTo: Entity? = nil) {
		self.speed = speed
		self.angle = initialAngle
		self.axis = axis
		self.relativeTo = relativeTo
	}
}

/// Works with the RevolvingComponent to apply a continuous rotation to an entity
@MainActor
class RevolvingSystem: System {
	private static let query = EntityQuery(where: .has(RevolvingComponent.self))
	
	required init(scene: RealityKit.Scene) {}
	
	func update(context: SceneUpdateContext) {
		for entity in context.entities(matching: Self.query, updatingSystemWhen: .rendering) {
			if var revolvingComponent = entity.components[RevolvingComponent.self] {
				let relativeTo = revolvingComponent.relativeTo
				
				revolvingComponent.angle += .pi * Float(context.deltaTime) * revolvingComponent.speed
				entity.setOrientation(.init(angle: revolvingComponent.angle, axis: revolvingComponent.axis), relativeTo: relativeTo)
				
				entity.components[RevolvingComponent.self] = revolvingComponent
			}
		}
	}
}

// MARK: Hover Control

/// Works with the HoverSystem to apply a continuous levitation like bounce to an entity
struct HoverComponent: Component {
	var speed: Float
	var angle: Float
	var from: SIMD3<Float>
	var to: SIMD3<Float>
	
	init(speed: Float = 0.06, angle: Float = 0, from: SIMD3<Float>, to: SIMD3<Float>) {
		self.speed = speed
		self.angle = angle
		self.from = from
		self.to = to
	}
}

/// Works with the HoverComponent to apply a continuous levitation like bounce to an entity
@MainActor
class HoverSystem: System {
	private static let query = EntityQuery(where: .has(HoverComponent.self))
	
	required init(scene: RealityKit.Scene) {}
	
	func update(context: SceneUpdateContext) {
		for entity in context.entities(matching: Self.query, updatingSystemWhen: .rendering) {
			if var hoverComponent = entity.components[HoverComponent.self] {
				
				hoverComponent.angle += .pi * Float(context.deltaTime) * hoverComponent.speed
				
				let range = hoverComponent.to - hoverComponent.from
				let proportion = (sin(hoverComponent.angle) + 1) / 2
				
				entity.position = hoverComponent.from + (proportion * range)
				
				entity.components[HoverComponent.self] = hoverComponent
			}
		}
	}
}

// MARK: Robot Parts

/// A wrapper around a Robot Character that is actually used as an Automaton
struct AutomatonControl: Component {
	var character: RobotCharacter
}

extension RobotCharacter {
	/// manually control the animation transition of a single robot instance
	func transitionToAndPlayAnimation(_ animationState: AnimationState) {
		if self.animationState.transition(to: animationState) {
			playAnimation(animationState)
		}
	}
}

/// A collection of shuffled robot names for our Automatons
@MainActor
enum RobotNames {
	static var count: Int = 0
	static var next: String {
		count += 1
		
		return "Robo-v\(count)"
	}
}

// MARK: Teleportation

/// Works with the TeleportationSystem to control spawning across all teleporters
struct ControlCenterComponent: Component {
	typealias SpawnHandler = (Entity) -> Void
	
	var initialValue: TimeInterval
	var interval: TimeInterval
	var countdown: TimeInterval
	var rootEntity: Entity
	var _spawnHandler: SpawnHandler
	
	init(initialValue: TimeInterval, interval: TimeInterval, rootEntity: Entity, spawnHandler: @escaping SpawnHandler) {
		self.initialValue = initialValue
		self.interval = interval
		self.countdown = initialValue
		self.rootEntity = rootEntity
		self._spawnHandler = spawnHandler
	}
}

/// Represents a single Teleporter in the TeleportationSystem
struct TeleporterComponent: Component {}

/// Works with the ControlCenterComponent to control spawning across all teleporters
@MainActor
class TeleportationSystem: System {
	private static let controlCenterQuery = EntityQuery(where: .has(ControlCenterComponent.self))
	private static let teleporterQuery = EntityQuery(where: .has(TeleporterComponent.self))
	private static let robotQuery = EntityQuery(where: .has(AutomatonControl.self))
	
	required init(scene: RealityKit.Scene) {}
	
	func update(context: SceneUpdateContext) {
		for entity in context.entities(matching: Self.controlCenterQuery, updatingSystemWhen: .rendering) {
			update(controlCenter: entity, context: context)
		}
	}
	
	private func safeToUse(teleporter: Entity, context: SceneUpdateContext) -> Bool {
		let someBotIsStandingToClose = context.entities(matching: Self.robotQuery, updatingSystemWhen: .rendering)
			.contains { entity in
				distance(entity.position(relativeTo: nil), teleporter.position(relativeTo: nil)) < 0.02
			}
		
		return  !someBotIsStandingToClose
	}
	
	private func update(controlCenter controlCenterEntity: Entity, context: SceneUpdateContext) {
		guard var controlCenter = controlCenterEntity.components[ControlCenterComponent.self],
			  let clubManager = controlCenter.rootEntity.components[DoorSupervisor.self],
			  clubManager.hasCapacity else {
			return
		}
		
		// 1. Decrease countdown, and activate if it reaches zero
		controlCenter.countdown -= context.deltaTime
		if controlCenter.countdown <= 0 {
			
			// 2. Find all the active teleporters and pick a random one
			if let teleporter = context.entities(matching: Self.teleporterQuery, updatingSystemWhen: .rendering).shuffled().first {
				
				// 3. If no other robots are in the way, pass it to the designated spawn method
				if safeToUse(teleporter: teleporter, context: context) {
					controlCenter._spawnHandler(teleporter)
				}
			}
			
			// 4. Set the delay till the next spawn event
			controlCenter.countdown = controlCenter.interval
		}
		
		// FIXME: Control Center is not being updated
	}
}

extension ParticleEmitterComponent.Presets {
	/// Makes a particle emitter component that looks like a teleporter
	fileprivate static var teleporter: ParticleEmitterComponent {
		var particleEmitter = ParticleEmitterComponent.Presets.rain
		
		particleEmitter.birthLocation = .surface
		particleEmitter.emitterShape = .torus
		particleEmitter.particlesInheritTransform = false
		particleEmitter.fieldSimulationSpace = .global
		particleEmitter.speed = 0.07
		particleEmitter.speedVariation = 0.03
		particleEmitter.radialAmount = 360
		particleEmitter.torusInnerRadius = 0.001
		particleEmitter.emissionDirection = [0, 1, 0]
		particleEmitter.spawnedEmitter = nil
		particleEmitter.burstCount = 5000
		particleEmitter.mainEmitter.opacityCurve = .linearFadeOut
		particleEmitter.mainEmitter.birthRate = 50
		particleEmitter.mainEmitter.birthRateVariation = 10
		particleEmitter.mainEmitter.lifeSpan = 0.5
		particleEmitter.mainEmitter.lifeSpanVariation = 0.01
		particleEmitter.mainEmitter.size = 0.001
		particleEmitter.mainEmitter.sizeVariation = 0.0005
		particleEmitter.mainEmitter.sizeMultiplierAtEndOfLifespan = 0.01
		particleEmitter.mainEmitter.stretchFactor = 10
		particleEmitter.mainEmitter.noiseStrength = 0
		particleEmitter.mainEmitter.spreadingAngle = 0
		particleEmitter.mainEmitter.angle = 0
		
		particleEmitter.spawnedEmitter = nil
		
		return particleEmitter
	}
}

// MARK: Dancing

/// Represents a single Attractor in the DanceMotivationSystem
struct AttractorComponent: Component {
	enum State {
		case vacant
		case attracting
		case motivating
	}
	
	private(set) var state: State = .vacant
	
	var target: Entity?
	var walkSpeed: Float = 0.1
	var interval: TimeInterval = 5
	var countdown: TimeInterval = 5
	var club: Entity?
	
	var isVacant: Bool {
		if case .vacant = state {
			return true
		}
		return false
	}
	
	mutating func setTarget(_ target: Entity) {
		self.target = target
		self.state = .attracting
	}
	
	mutating func targetReached() {
		self.state = .motivating
	}
}

/// Represents a single Robot in the DanceMotivationSystem
struct Newcomer: Component {}

/// Works with the DanceMotivationSystem to provide additional Debug information to the RealityKit Debugger
struct DanceSystemDebugComponent: Component {
	var states: UIImage? = nil
	var vacant: Int = 0
	var attracting: Int = 0
	var motivating: Int = 0
}

/// Provides additional Debug information about a single Attractor in the DanceMotivationSystem to the RealityKit Debugger
struct AttractorDebugComponent: Component {
	var state: AttractorComponent.State
	var attractor: Entity
	var robot: Entity?
}

/// Manages the states of dance floor attractors, the movement of robots and the relationships between them
@MainActor
class DanceMotivationSystem: System {
	private static let attractorQuery = EntityQuery(where: .has(AttractorComponent.self))
	private static let targetQuery = EntityQuery(where: .has(Newcomer.self))
	private static let clubbersQuery = EntityQuery(where: .has(AutomatonControl.self))
	private static let debugRootQuery = EntityQuery(where: .has(DanceSystemDebugComponent.self))
	private static let debugVisualizationsQuery = EntityQuery(where: .has(AttractorDebugComponent.self))
	
	required init(scene: RealityKit.Scene) {}
	
	func update(context: SceneUpdateContext) {
		
		// 1. Check for newcomers at the club who could be enticed to come and dance
		for visitor in context.entities(matching: Self.targetQuery, updatingSystemWhen: .rendering) {
			
			// 2. Randomly pick an attractor
			guard let attractor = context.entities(matching: Self.attractorQuery, updatingSystemWhen: .rendering)
				.filter({ $0.components[AttractorComponent.self]?.isVacant ?? false })
				.randomElement() else {
				return
			}
			
			// 3. Start attracting the visitor
			var attractorComponent = attractor.components[AttractorComponent.self]!
			attractorComponent.setTarget(visitor)
			attractor.components[AttractorComponent.self] = attractorComponent
			
			// FIXME: Stop attractors competing over the same bot
		}
		
		// Let the attractors do their thing and attract visitors to come and dance
		for attractor in context.entities(matching: Self.attractorQuery, updatingSystemWhen: .rendering) {
			guard var attractorComponent = attractor.components[AttractorComponent.self] else {
				continue
			}
			
			switch attractorComponent.state {
			case .attracting:
				if let updatedAttractorComponent = attractRobot(attractor: attractor, deltaTime: Float(context.deltaTime)) {
					attractorComponent = updatedAttractorComponent
				}
				
			case .motivating:
				if let updatedAttractorComponent = motivateRobot(attractor: attractor, context: context) {
					attractorComponent = updatedAttractorComponent
				}
				
			default:
				break
			}
			
			// save changes
			attractor.components[AttractorComponent.self] = attractorComponent
		}
		
#if DEBUG
		updateDebugInfo(context: context)
#endif
	}
	
	private func attractRobot(attractor: Entity, deltaTime: Float) -> AttractorComponent? {
		guard var attractorComponent = attractor.components[AttractorComponent.self],
			  case .attracting = attractorComponent.state,
			  let target = attractorComponent.target,
			  let robotCharacter = target.components[AutomatonControl.self]?.character else {
			return nil
		}
		
		// robots wave when they first arrive, make sure that is completed first before moving
		var transitionAnimationTo: AnimationState?
		switch robotCharacter.animationState {
		case .wave: transitionAnimationTo = .idle
		case .idle: transitionAnimationTo = .walkLoop
		case .walkLoop: transitionAnimationTo = nil
		default: return attractorComponent
		}
		
		if let transitionAnimationTo {
			if robotCharacter.animationState.transition(to: transitionAnimationTo) {
				robotCharacter.playAnimation(robotCharacter.animationState)
			}
		}
		
		// Convert the robot and target positions into the same coordinate system
		let targetPosition = target.position(relativeTo: attractorComponent.club)
		var danceSpotPosition = attractor.position(relativeTo: attractorComponent.club)
		danceSpotPosition.y = targetPosition.y
		
		let movementVector = danceSpotPosition - targetPosition
		let normalizedMovement = movementVector / length(movementVector)
		let move = normalizedMovement * deltaTime * attractorComponent.walkSpeed
		
		target.setPosition(targetPosition + move, relativeTo: attractorComponent.club)
		
		robotCharacter.characterModel.look(at: robotCharacter.characterModel.position - normalizedMovement,
										   from: robotCharacter.characterModel.position, relativeTo: robotCharacter.characterParent)
		
		// If the target is more or less in position then attach to the dance spot and change state to motivating
		if distance(danceSpotPosition, target.position(relativeTo: attractorComponent.club)) < 0.005 {
			attractor.addChild(target, preservingWorldTransform: true)
			
			// Start Dancing
			robotCharacter.transitionToAndPlayAnimation(.celebrate)
			
			// Update attractor state
			attractorComponent.targetReached()
		}
		
		return attractorComponent
	}
	
	private func motivateRobot(attractor: Entity, context: SceneUpdateContext) -> AttractorComponent? {
		guard var attractorComponent = attractor.components[AttractorComponent.self],
			  case .motivating = attractorComponent.state,
			  let target = attractorComponent.target,
			  let robotCharacter = target.components[AutomatonControl.self]?.character else {
			return nil
		}
		
		attractorComponent.countdown -= context.deltaTime
		
		if attractorComponent.countdown <= 0 {
			// Turn to face a random fellow clubber
			if let friend = Array(context.entities(matching: Self.clubbersQuery, updatingSystemWhen: .rendering)).randomElement() {
				let friendsPosition = friend.position(relativeTo: robotCharacter.characterParent)
				
				robotCharacter.characterModel.look(at: friendsPosition,
												   from: robotCharacter.characterModel.position, relativeTo: robotCharacter.characterParent)
				
				// TODO: remove me
				print("🔥 friendsPosition \(friendsPosition) targetPosition \(robotCharacter.characterModel.position)")
			}
			
			attractorComponent.countdown = attractorComponent.interval
		}
		
		return attractorComponent
	}
	
#if DEBUG
	let vacantColor = UnlitMaterial.BaseColor(tint: .yellow.withAlphaComponent(0.5))
	let attractingColor = UnlitMaterial.BaseColor(tint: .orange.withAlphaComponent(0.5))
	let motivatingColor = UnlitMaterial.BaseColor(tint: .red.withAlphaComponent(0.5))
	
	private func updateDebugInfo(context: SceneUpdateContext) {
		var vacantCount: Int = 0
		var attractingCount: Int = 0
		var motivatingCount: Int = 0
		
		context.entities(matching: Self.debugVisualizationsQuery, updatingSystemWhen: .rendering).forEach { visualization in
			guard let visualizationComponent = visualization.components[AttractorDebugComponent.self],
				  let attractorComponent = visualizationComponent.attractor.components[AttractorComponent.self] else {
				return
			}
			
			updateVisualizationEntity(visualization, relativeTo: attractorComponent.club)
			
			switch attractorComponent.state {
			case .vacant: vacantCount += 1
			case .attracting: attractingCount += 1
			case .motivating: motivatingCount += 1
			}
		}
		
		context.entities(matching: Self.debugRootQuery, updatingSystemWhen: .rendering).forEach { debugRoot in
			if var debugComponent = debugRoot.components[DanceSystemDebugComponent.self] {
				debugComponent.vacant = vacantCount
				debugComponent.attracting = attractingCount
				debugComponent.motivating = motivatingCount
				debugComponent.states = makeChart(vacantCount: vacantCount, attractingCount: attractingCount, motivatingCount: motivatingCount)
				debugRoot.components[DanceSystemDebugComponent.self] = debugComponent
			}
		}
	}
	
	private func updateVisualizationEntity(_ visualization: Entity, relativeTo root: Entity?) {
		guard var visualizationComponent = visualization.components[AttractorDebugComponent.self],
			  let attractorComponent = visualizationComponent.attractor.components[AttractorComponent.self] else {
			return
		}
		
		// Update the position
		var position = visualizationComponent.attractor.position(relativeTo: root)
		position.y = visualization.position.y
		visualization.setPosition(position, relativeTo: root)
		
		// Update the state
		visualizationComponent.state = attractorComponent.state
		visualization.name = "[Debug] \(visualizationComponent.attractor.name) (\(attractorComponent.state))"
		
		// Update the base material color to signify the attractor state
		if var modelComponent = visualization.components[ModelComponent.self],
		   var material = modelComponent.materials.first as? UnlitMaterial {
			
			switch attractorComponent.state {
			case .vacant: material.color = vacantColor
			case .attracting: material.color = attractingColor
			case .motivating: material.color = motivatingColor
			}
			
			modelComponent.materials = [material]
			visualization.components[ModelComponent.self] = modelComponent
		}
		
		// Update the target
		visualizationComponent.robot = attractorComponent.target
		visualization.components[AttractorDebugComponent.self] = visualizationComponent
	}
	
	private func makeChart(vacantCount: Int, attractingCount: Int, motivatingCount: Int) -> UIImage? {
		ImageRenderer(content: chartView(vacantCount: vacantCount, attractingCount: attractingCount, motivatingCount: motivatingCount)).uiImage
	}
	
	private func chartView(vacantCount: Int, attractingCount: Int, motivatingCount: Int) -> some View {
		Chart(
			[
				(name: "Vacant", count: vacantCount),
				(name: "Attracting", count: attractingCount),
				(name: "Motivating", count: motivatingCount)
			], id: \.name) { name, count in
				SectorMark(
					angle: .value("Value", count),
					angularInset: 1.5
				)
				.cornerRadius(5)
				.foregroundStyle(by: .value("Name", name))
		}
		.chartLegend(.hidden)
		.chartForegroundStyleScale(["Vacant": .yellow, "Attracting": .orange, "Motivating": .red])
		.frame(width: 1024, height: 1024)
	}
	
#endif
}

// MARK: Debug Helpers

extension Entity {
	/// creates an semi-transparent entity that can be useful in debug invisible entities in the RealityKit Debugger
	static func makeDebugMarker(name: String? = nil, height: Float, radius: Float, color: UIColor = .white, enabled: Bool = false) -> Entity? {
#if DEBUG
		var debugMaterial = UnlitMaterial()
		debugMaterial.color = .init(tint: color)
		debugMaterial.blending = .transparent(opacity: 0.7)
		
		let marker = ModelEntity(mesh: .generateCylinder(height: height, radius: radius), materials: [debugMaterial])
		if let name {
			marker.name = name
		}
		marker.isEnabled = enabled
		
		return marker
#else
		return nil
#endif
	}
	
	/// adds an semi-transparent child entity that can be useful in debug invisible entities in the RealityKit Debugger
	@discardableResult
	func addDebugMarker(name: String? = nil, height: Float? = nil, radius: Float? = nil, color: UIColor = .white, enabled: Bool = false) -> Entity? {
#if DEBUG
		var markerRadius: Float
		if radius != nil {
			markerRadius = radius!
		} else {
			// If no provided radius then calculate from the visual bounds
			let extents = visualBounds(relativeTo: nil).extents
			let boundingXZRadius = max(extents.x, extents.z) / 2
			
			if boundingXZRadius.isNormal {
				markerRadius = boundingXZRadius
			} else {
				// If no visual bounds then use a default radius of 1cm
				markerRadius = 0.01 * scale(relativeTo: nil).max()
			}
		}
		
		// If no provided height then use a default value of 10cm
		let markerHeight = height ?? 0.1 * scale(relativeTo: nil).max()
		
		let name = name ?? "[Debug] \(self.name)"
		if let marker = Entity.makeDebugMarker(name: name, height: markerHeight, radius: markerRadius, color: color, enabled: enabled) {
			marker.position = [0, markerHeight / 2, 0]
			addChild(marker)
			
			return marker
		}
#endif
		return nil
	}
}

// MARK: Demo Helpers

extension MeshResource {
	/// Generates an cylinder with all the normals facing downwards. Probably has no uses other than demo'ing a broken mesh.
	static func generateAbnormalCylinder(height: Float, radius: Float) -> MeshResource {
		let meshResource = MeshResource.generateCylinder(height: height, radius: radius)
		var contents = meshResource.contents
		let models = contents.models.map { model in
			var model = model
			let parts = model.parts.map { part in
				var part = part
				part.normals = part.normals.map { normals in
					let transformedNormals: [SIMD3<Float>] = normals.map { _ in
						[0, -1, 0]
					}
					
					return MeshBuffer(transformedNormals)
				}
				
				return part
			}
			model.parts = MeshPartCollection(parts)
			
			return model
		}
		contents.models = MeshModelCollection(models)
		try? meshResource.replace(with: contents)
		
		return meshResource
	}
}
```
### Add a volumetric club scene - 3:02
```swift
WindowGroup(id: "RobotClub") {
    GeometryReader3D { geometry in
        ClubView()
            .volumeBaseplateVisibility(.visible)
            .environment(appState)
            .scaleEffect(geometry.size.width / initialVolumeSize.width)
    }
    .onAppear {
        dismissWindow(id: "RobotCreation")
    }
}
.windowStyle(.volumetric)
.defaultWorldScaling(.dynamic)
.defaultSize(initialVolumeSize)
```

### Add a button to open the club - 3:09
```swift
VStack {
    Button("🪩") {
        openWindow(id: "RobotClub")
    }
    .padding()
    
    Spacer()
}
.padding([.trailing, .top])
```

### FIX: Unintentionally inheriting an ancestor's transformation - 6:50
```swift
discoBall.addChild(background)
```

### FIX: Control Center is not being updated - 10:18
```swift
// 5. Save updated component back to the entity
controlCenterEntity.components[ControlCenterComponent.self] = controlCenter
```

### FIX: Stocking bottles - 18:15
```swift
private func stockBottles(placementRadius: Float) -> Entity {
    let bottleRadius: Float = 0.003
    let bottleHeight: Float = 0.022
    let angleIncrement: Float = -12
    let outOfStockBrands: Set = [3]
    
    // Make a wrapper entity
    let bottleGroup = Entity()
    bottleGroup.name = "Bottle Group"
    bottleGroup.position = [0, 0.04, 0]
    bottleGroup.orientation = .init(angle: 180 * (.pi / 180), axis: [0, 1, 0])
    
    // Make a nice green material
    var bottleMaterial = PhysicallyBasedMaterial()
    bottleMaterial.baseColor = .init(tint: .green)
    bottleMaterial.blending = .transparent(opacity: .init(floatLiteral: 0.5))
    
    for i in 0..<9 {
        let angle = Angle2D(degrees: angleIncrement * Float(i))
        let bottleMesh = MeshResource.generateCylinder(height: bottleHeight, radius: bottleRadius)
        let bottle = ModelEntity(mesh: bottleMesh, materials: [bottleMaterial])
        bottle.name = "BT\(i)"
        bottle.position = pointOnCircumference(angle: angle, radius: placementRadius, y: bottleHeight / 2)
        if outOfStockBrands.contains(i) {
            bottle.components[OutOfStockComponent.self] = OutOfStockComponent()
        }
        
        bottleGroup.addChild(bottle)
    }
    
    return bottleGroup
}
```

### FIX: Attractors - 22:48
```swift
// 4. Untag them as a Newcomer
visitor.components[Newcomer.self] = nil
```