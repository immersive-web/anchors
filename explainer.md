# What Is An Anchor?

An anchor is a concept that allows applications to specify poses, a position and orientation in three dimensional space, that will be tracked by the underlying system. There are systems where pose tracking is based on their understanding of the world. That understanding, and thus the pose of an anchor, varies over time. Anchors allow a developer to specify poses in the world that need to be updated to correctly reflect the evolving understanding of the world, such that the poses remain aligned with the same place in the physical world.

Augmented Reality systems are examples of systems that are constantly evolving their understanding of the world, both the understanding of the user pose (via tracking a mobile device or HMD) as well as the understanding of the physical structure of the space around the user, real world objects like planar targets and faces, or semantic understanding of objects like cars or tables. Anchors allow developers to specify that a pose is intended to remain aligned in three dimensional space relative to something in the physical world. 

The main idea behind the concept of an anchor is that as the underlying platform's understanding of the world evolves over time, the pose will be updated such that it remains aligned with the same place in the physical world.

# Terminology

*   **Pose**: a three dimensional position and orientation in a three dimensional coordinate system. It can be represented in many different forms but the most common ones are usually a 4x4 transformation matrix or a 3 value vector for the position and a 4 value quaternion for the orientation.
*   **Anchor**: a concept that allows developers to specify a pose that can change over time, either because the system's understanding of the physical world coordinates changes or because the understanding of an object in the world the anchor is relative to changes.
    *   **Important note about the concept of anchor in Apple's iOS [ARKit](https://developer.apple.com/documentation/arkit) and in this explainer:** iOS ARKit's SDK uses the concept of an anchor to represents 2 ideas at the same time:
        1.  An arbitrary pose in 3D space that needs to be updated relative to the physical world. Anchors that represent an arbitrary pose can be created and registered in ARKit. This is the same as the anchor concept in this explainer.
        1.  A real world object that the system is able to identify from the real world understanding. These elements have a pose, but also include other information such as geometry. At the moment of the publication of this explainer ARKit is able to understand [ARPlaneAnchor](https://developer.apple.com/documentation/arkit/arplaneanchor), [ARFaceAnchor](https://developer.apple.com/documentation/arkit/arfaceanchor) and [ARImageAnchor](https://developer.apple.com/documentation/arkit/arimageanchor) as anchors.

        While ARKit uses the concept of an anchor to represent the pose and the identified real world object, this explainer currently uses the term anchor to only represent an object with a pose that specifies its location relative to the physical world. The [ARAnchor](https://developer.apple.com/documentation/arkit/aranchor) (1) base class in ARKit would be equivalent of the concept of an anchor in this explainer. Additional information about the representation of real world objects is out of the scope of this explainer. This differentiation between the concept of an anchor in ARKit and in the scope of this explainer is subtle but important.


# Scope

Anchors can represent different concepts:


1.  An object with an arbitrary three dimensional pose that must be updated as the understanding of the world coordinate system evolves.
2.  An object with a pose relative to a specific real world object the system has been able to identify and track.

Anchors could also represent entities with poses that:

3.  Persist, meaning that the anchor is able to live between executions of the same application.
4.  Are shared between different applications.

The two uses are currently out of the scope of the anchor proposal.

This explainer focuses on the first two concepts of anchors: as specifying the pose of a location in the world (1), and as establishing a relationship to semantically meaningful parts of the physical real world that the system has detected (2). 

Creating anchors in relation to the structure of the physical world around the user (such as if the system supports intersecting a ray with the system's understanding of the physical world) is expected to be a very common practice.

The reasons for this limited scope are:

*   Arbitrary anchors are the most basic yet useful type of anchors. There is always a need, no matter the granularity of the real world understanding the system has, to create arbitrary 3D poses as anchors. Moreover, anchors that represent arbitrary 3d poses can be used to represent positions relative to detected or tracked objects when world understanding is expanded to include object detection and tracking.
*   Persisting and sharing anchors is outside of the current scope of this explainer, since platform-level anchors (used to implement the anchor concept) are opaque and not compatible across different platforms.

Anchors are intended to maintain a pose that corresponds to a location in the physical world, and will be updated by the system as its understanding of the physical-world changes. Anchors are currently **not** intended to track moving objects.

This document assumes that WebXR supports hit-testing API as described by the [hit-testing explainer](https://github.com/immersive-web/webxr/blob/master/hit-testing-explainer.md). Real-world geometry (RWG) detection APIs are currently incubated in its own [repository](https://github.com/immersive-web/real-world-geometry/).

# Use Cases

In general, applications should use anchors whenever a virtual object is placed in the scene. This is the only way to ensure the pose of the virtual object will be continuously updated to maintain a fixed relationship (i.e., position/orientation) with the physical world over time.

There are different strategies an application may use to create anchors, depending on its particular scene graph structure, but as a general use case, any virtual object that will be positioned in world space should be positioned relative to an anchor. In some cases, a single anchor might serve as the base coordinate system for multiple objects, such as when an app places a complex scene composed of multiple elements somewhere in the physical world. Only one anchor is needed in this case, as all of the elements are relative to the same physical location.

Imagine a small race track with cars on it. The race track will be placed at a location world space, so an anchor should be created for it, with  the cars positioned relative to the race track. A pose update for the race track's anchor will update the location of the race track and, in turn, the location of the cars position relative to it.

If the race track is very large, and has been laid out interactively by the user pointing at locations in the world, multiple anchors might be used for each of these locations.  In this case, an application might need to adjust the exact model of the track over time, as anchors might shift relative to each other as the user moves around the exact locations of the anchors change.  Such a scenario is more complex, but will ensure that different parts of the track are anchored to the locations the user specified, despite the system's understanding of the world evolving.

Although most use cases for anchor creation might be related to real-world understanding (e.g., placing the virtual race track on top of a real table), there are also use cases where anchors are created based on an arbitrary pose relative to the user's head (i.e., the system camera), instead of a direct relationship to the physical world. Imagine placing a virtual HUD (Heads Up Display) in mid air in front of the user to access a quick menu in AR. The placement of the UI element could be at some specific comfortable distance in front of the user, and an anchor should be used to make sure the UI stays at the correct position/orientation even if the world coordinates change while the menu is visible.

Two examples where Anchors might update as real-world understanding improves are:

1.  The system gains a more precise understanding of where a physical object is, which affects the pose created relative to it. For example, if a pose was created relative to a plane perceived at 1m above the floor, and as the user moves around the system refines this estimate to being 0.95m above the floor, then the Anchor pose will be updated;
1.  The system shifts the world coordinate system used to specify the camera location. For example, if a pose was created (relative to the physical world, or to the camera) such that the world position is (1,1,1), and the system subsequently shifts its world coordinates such that the position that was formerly (1,1,1) is now (0.75,1,1), then the Anchor pose must be updated accordingly.


# Possible API Considerations

*   Although anchors might be mostly created in relation to real world understanding elements (result of a hit test of a ray with the physical world, for example), there must always be a way to create an anchor at any arbitrary pose in space.
*   Because it has major implications both in terms of performance and the future requirements of concepts like sharing and persistency, anchors should be created in an asynchronous way.
*   As WebXR introduces the concept of coordinate systems in anything related to poses, and anchors are created in relation to poses, it is reasonable to assume that the anchor API should have a strong coupling to coordinate systems.
*   As the most common use case for anchors is to attach a virtual object to a location in the physical world based on the underlying system's understanding of the world, anchors might need to be tightly coupled to world understanding APIs inside the WebXR Device API. For example, an anchor should be created when placing a virtual object on the pose provided by a plane or hit test result. Coordinating the development of anchors with such APIs seems like a good idea.
*   Anchors can lose tracking without application's intervention - there needs to be a way to notify the application of the fact that an anchor is no longer tracked by the system.


## Add anchor - API Details

`addAnchor` parameters
*   pose - the initial pose where the anchor should be created. The system will make sure that the relationship with the physical world made at this moment in time is maintained as the tracking system evolves.
*   referenceSpace - the frame of reference the pose is relative to.

To enable feature detection of possible future versions of the API with additional parameters, an error is thrown if additional arguments are given to the function.

`addAnchor` return value
*   Returns a Promise<XRAnchor>. The meaning of the promise resolution is as follows:
    *   If the promise rejects, there was an error (should be detailed in a returned string). It could be that the API is unsupported by the platform or the XRSession is not of the right type for creating anchors. It could be an internal error of some kind.
    *   If the promise resolves, the anchor was successfully added to the underlying system and a valid XRAnchor should be provided.
*   As this function returns a promise, the actual XRAnchor will be provided to the application in the near future. In the case of anchors, the creation of the anchor should happen in the next frame and before the request animation frame of the session is called, if called during an animation frame callback. If called outside of an animation callback, the promise might resolve before the next request animation frame, but may not. The promise should provide an XRAnchor whose internal pose should always be up to date. For that reason, the app should be aware that it could be possible that the pose of the XRAnchor to be different than the original pose passed to create the anchor, and to possibly change each frame.
*   XRAnchor.anchorSpace can be used to obtain an anchor pose that is always up to date. The app should always update any virtual objects that should be located relative to the anchor, using the value of this pose.

## Code examples

The following code examples try to clarify the proposed IDL API.

### Adding anchors

```javascript
var anchorToModelMap = new Map();
// Create an arbitrary anchor
session.addAnchor(anchorPose, eyeLevelReferenceSpace).then((anchor) => {
  // Somehow retrieve the virtual object model that will be related to the anchor.
  let model = ...;
  anchorToModelMap.put(anchor, model);
}, (error) {
  console.error(“Could not create arbitrary anchor: “ + error);
});
```

### Removing anchors

```javascript
for(var anchor of anchorToModelMap.keys()) {
  anchor.detach();
}
anchorToModelMap.clear();
```

### Updating anchors

```javascript
  anchor.addEventListener(“update”, (event) => {
    var anchor = event.source;
    var model = anchorToModelMap.get(anchor);

    // Update the position of the model based on current pose of the anchor.
  });
```
