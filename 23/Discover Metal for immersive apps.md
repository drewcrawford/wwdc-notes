Find out how you can use Metal to render fully immersive experiences for visionOS. We'll show you how to set up a rendering session on the platform and create a basic render loop, and share how you can make your experience interactive by incorporating spatial input.

Create immersive experiences with familiar technologies.   Blend virtual content with the real world.  If your application will take the user into a fully immersive experience, also completely replace with own virtual content.

You have choices.  RealityKit?  Or metal and RK APIs.

# App architecture
developer.apple.com/Metal

Conform to swiftui app.  Define a list of scenes.  3 scene types
* window -> experience similar to 2d platforms
* volume -> renders content within its bounds, shared space, etc.
* immersive -> render content anywhere

ImmersiveSpace
 * conforms to swiftui scene
 * contauiner for immersive experiences
* [[Go beyond the window with SwiftUI]]

when you create an immersivespace scene, your app provides content to conform to `ImmersiveSpaceContent` protocol.  Often, application will use RK.  Uses coreanimation and materialx.  But instead, you can use metal.

CompositorServices API uses metal and arkit to provide immersive rendering capabilities.  
Metal rendering interface.
render directly to the compositor server
low ipc overhead to minimize latency
supports c and swift apis

CompositorLayer.  Provide 2 paramteters.
* compositorlayerconfiguration.  Defines behavior and capabilities of rendering session
* layerrenderer -> interface to rendering session.  app uses this object to schedule/render a new frame.


```swift
@main
struct MyApp: App {
    var body: some Scene {
        ImmersiveSpace {
            CompositorLayer { layerRenderer in
                let engine = my_engine_create(layerRenderer)
                let renderThread = Thread {
                    my_engine_render_loop(engine)
                }
                renderThread.name = "Render Thread"
                renderThread.start()
            }
        }
    }
}
```

swiftui creates a bounded window by default.

modify UIApplicationPreferredDefaultSceneSessionRole to CPSceneSessionRoleImmersiveSpaceApplication.

# Render configuration

to provide a configuration to compositor layer, create new type conforming to `CompositorLayerConfiguration` protocol.  Allows yo tyou modify setup, etc.

## layer configuration
* layerRenderer capabilities
	* query what features are available on device.
* LayerRenderer Configuration
	* foveated rendering
	* layerrenderer layout
	* color management

### foveated rendering
higher pixel per degree density.  In a regular display pipeline, pixels are distributed linearly in a texture.  xrOS optimizes this workflow by creating a map to use lower sampling rate in certain regions.  Reduce the power required to render frame.

Using this is important, results in better visual experience.  Use xcode's metal debugger.  Inspect target textures and rasterization rate maps, etc.

MTLRasterizationRatemap
Check if foveation is supported `capabilities.supportsFoveation`.  
set if your application supports foveation.

### layout
each eye first maps into a mtltexture provided by compositor.  Slice index.  Viewport.  

choose different mappings between texture, slice, and viewport.  

* layered -> one texture, with 2 slices and two viewports
* dedicated -> two textures, one slice, one viewport
* shared -> one texture, one slice, 2 viewports.


layered and shared -> perform rendering in one single pass.
shared -> might be easier to port existing codebases where vobeated is not an option

layered -> optimal.  Render in a single pass while still maintaining foveated rendering.

### color management
`.extendedLInearDisplayP3`
we support EDR headroom of 2.0
HDR-copmatible pixel format.
If your application supports HDR, specify rgba16float in layer configuration

[[Explore HDR rendering with EDR]]
### CompositorLayer Configuration - 10:32
```swift
// CompositorLayer configuration

struct MyConfiguration: CompositorLayerConfiguration {
    func makeConfiguration(capabilities: LayerRenderer.Capabilities,
                           configuration: inout LayerRenderer.Configuration) {

        let supportsFoveation = capabilities.supportsFoveation
        let supportedLayouts = capabilities.supportedLayouts(options: supportsFoveation ?
                                                             [.foveationEnabled] : [])

        configuration.layout = supportedLayouts.contains(.layered) ? .layered : .dedicated

        configuration.isFoveationEnabled = supportsFoveation

        // HDR support
        configuration.colorFormat = .rgba16Float
   }
}
```
# Render loop
Setup render loop.
Check LayerRenderer state
if paused, wait until running.
check state again.  If running, render frame.  Then check layer state again.

If layer state is invalidated, free render loop resources.

Note that immersive space api is only available in swift.  
### Render loop - 12:20
```swift
void my_engine_render_loop(my_engine *engine) {
    my_engine_setup_render_pipeline(engine);

    bool is_rendering = true;
    while (is_rendering) @autoreleasepool {
        switch (cp_layer_renderer_get_state(engine->layer_renderer)) {
            case cp_layer_renderer_state_paused:
                cp_layer_renderer_wait_until_running(engine->layer_renderer);
                break;
            case cp_layer_renderer_state_running:
                my_engine_render_new_frame(engine);
                break;
            case cp_layer_renderer_state_invalidated:
                is_rendering = false;
                break;
        }
    }

    my_engine_invalidate(engine);
}
```

## ARKIt
New xros api
tracking session for world, hands, more.
C/Swift APIs

[[Meet ARKit for spatial computing]]


# Render a frame

1.  Update.
	2. Submission.  Latency-critical work, headset pose dependent, etc.

time is critical.

1.  Optimal input time.
2. Rendering deadline.
3. Presentation time.

so the flow is

1.  Update
2. wait for input time
3. submission
4. GPU renders.
5. compositor does stuff.


### Render new frame - 15:56
```swift
void my_engine_render_new_frame(my_engine *engine) {
    
    cp_frame_t frame = cp_layer_renderer_query_next_frame(engine->layer_renderer);
    if (frame == nullptr) { return; }
    
    cp_frame_timing_t timing = cp_frame_predict_timing(frame);
    if (timing == nullptr) { return; }

    cp_frame_start_update(frame);

    my_input_state input_state = my_engine_gather_inputs(engine, timing);
    my_engine_update_frame(engine, timing, input_state);

    cp_frame_end_update(frame);

    // Wait until the optimal time for querying the input
    cp_time_wait_until(cp_frame_timing_get_optimal_input_time(timing));

    cp_frame_start_submission(frame);

    cp_drawable_t drawable = cp_frame_query_drawable(frame);
    if (drawable == nullptr) { return; }

    cp_frame_timing_t final_timing = cp_drawable_get_frame_timing(drawable);
    ar_pose_t pose = my_engine_get_ar_pose(engine, final_timing);
    cp_drawable_set_ar_pose(drawable, pose);

    my_engine_draw_and_submit_frame(engine, frame, drawable);

    cp_frame_end_submission(frame);
}
```

Note we have C apis for all this stuff, not objc.

# User input

LayerREndering -> get updates every time the app receives a pinch event.  Exposed in the form of spatial events.  3 main properties.
1.  Phase.  Is event active, finished, cancelled?
2. selection ray -> determine content of the scene that had the attention
3. manipulator pose -> pose of the pinch, and gets updated every frame for event duration.

from hand tracking, can get left/right hand skeletal info, etc.

decide between hand rendering or passthrough hands.

### App architecture + input support - 18:57
```swift
@main
struct MyApp: App {
    var body: some Scene {
        ImmersiveSpace {
            CompositorLayer(configuration: MyConfiguration()) { layerRenderer in
                let engine = my_engine_create(layerRenderer)
                let renderThread = Thread {
                    my_engine_render_loop(engine)
                }
                renderThread.name = "Render Thread"
                renderThread.start()
                layerRenderer.onSpatialEvent = { eventCollection in
                    var events = eventCollection.map { my_spatial_event($0) }
                    my_engine_push_spatial_events(engine, &events, events.count)
                }
            }
        }
        .upperLimbVisibility(.hidden)
    }
}
```

### Push spatial events - 18:57
```swift
void my_engine_push_spatial_events(my_engine *engine,
                                   my_spatial_event *spatial_event_collection,
                                   size_t event_count) {
    os_unfair_lock_lock(&engine->input_event_lock);
    
    // Copy events into an internal queue
    
    os_unfair_lock_unlock(&engine->input_event_lock);
}
```

Note that updates are delivered in main thread.  Use some synchronization mechanism when reading/writing events in your engine.


### Gather inputs - 19:57
```swift
my_input_state my_engine_gather_inputs(my_engine *engine,
                                       cp_frame_timing_t timing) {
    my_input_state input_state = my_input_state_create();

    os_unfair_lock_lock(&engine->input_event_lock);
    input_state.current_pinch_collection = my_engine_pop_spatial_events(engine);
    os_unfair_lock_unlock(&engine->input_event_lock);

    ar_hand_tracking_provider_get_latest_anchors(engine->hand_tracking_provider,
                                                 input_state.left_hand,
                                                 input_state.right_hand);

    return input_state;
}
```

# Wrap up
* define the app swith swiftui
* render compsitorservices and metal
* add interactivity with ARKit.

# Resources
* https://developer.apple.com/documentation/compositorservices/drawing_fully_immersive_content_using_metal
* https://developer.apple.com/documentation/metal
