# What Is An Anchor?

An anchor is a concept that allows applications to specify poses, a position and orientation in three dimensional space, that will be tracked by the underlying system. There are systems where pose tracking is based on their understanding of the world. That understanding, and thus the pose of an anchor, varies over time. Anchors allow a developer to specify poses in the world that need to be updated to correctly reflect the evolving understanding of the world, such that the poses remain aligned with the same place in the physical world.

Augmented Reality systems are examples of systems that are constantly evolving their understanding of the world, both the understanding of the user pose (via tracking a mobile device or HMD) as well as the understanding of the physical structure of the space around the user, real world objects like planar targets and faces, or semantic understanding of objects like cars or tables. Anchors allow developers to specify that a pose is intended to remain aligned in three dimensional space relative to something in the physical world. 

The main idea behind the concept of an anchor is that as the underlying platform's understanding of the world evolves over time, the pose will be updated such that it remains aligned with the same place in the physical world.

# Terminology

*   **Pose**: a three dimensional position and orientation in a three dimensional coordinate system. It can be represented in many different forms but the most common ones are usually a 4x4 transformation matrix or a 3 value vector for the position and a 4 value quaternion for the orientation.
*   **Anchor**: a concept that allows developers to specify a pose that can change over time either because the system's understanding of the physical world coordinates changes, or because the understanding of an object in the world the anchor is relative to changes.
    *   **Important note about the concept of anchor in Apple's iOS [ARKit](https://developer.apple.com/documentation/arkit) and in this explainer:** iOS ARKit's SDK uses the concept of an anchor to represents 2 ideas at the same time:
        1.  An arbitrary pose in 3D space that needs to be updated relative to the physical world. Anchors that represent an arbitrary pose can be created and registered in ARKit. This is the same as the anchor concept in this explainer.
        1.  A real world object that the system is able to identify from the real world understanding. These elements have have a pose, but also include other information such as geometry. At the moment of the publication of this explainer ARKit is able to understand [ARPlaneAnchor](https://developer.apple.com/documentation/arkit/arplaneanchor), [ARFaceAnchor](https://developer.apple.com/documentation/arkit/arfaceanchor) and [ARImageAnchor](https://developer.apple.com/documentation/arkit/arimageanchor) as anchors.

        While ARKit uses the concept of an anchor to represent the pose and the identified real world object, this explainer currently uses the term anchor to only represent an object with a poses that specifies it's location relative to the physical world. The [ARAnchor](https://developer.apple.com/documentation/arkit/aranchor) (1) base class in ARKit would be equivalent of the concept of an anchor in this explainer. Additional information about the representation of real world objects is out of the scope of this explainer. This differentiation between the concept of an anchor in ARKit and in the scope of this explainer is subtle but important.


# Scope

Anchors can represent different concepts:


1.  An object with an arbitrary three dimensional pose that must be updated as the understanding of the world coordinate system evolves.
2.  An object with a pose relative to a specific real world object the system has been able to identify and track.

Anchors could also represent entities with poses that:

3.  Persist, meaning that the anchor is able to live between executions of the same application.
4.  Are shared between different applications.

This explainer focuses on just the first concept of anchors (1) as specifying the pose of a location in the world, but leaves open the possibility for anchors knowing their relationship to semantically meaningful parts of the physical real world that the system has detected (2). 

Creating anchors in relation to the structure of the physical world around the user (such as if the system supports intersecting a ray with the the systems understanding of the physical world) is a very common practice practice that we want to support.

The reasons for this limited scope are:

*   Arbitrary anchors are the most basic yet useful type of anchors. There is always a need, no matter the granularity of the real world understanding the system has, to create arbitrary 3D poses as anchors. Moreover, arbitrary 3d pose anchors can be used to represent positions relative to detect or tracked objects when world understanding is expended to include object detection and tracking.
*   Understanding the semantics of the physical world (such as detecting and tracking markers, images, faces or objects) is outside the scope of this explainer. Currently, there is no specific proposal on how to expose even basic understanding of the physical world to the web, much less more interesting objects. Whenever such proposal is agreed upon, the scope of the anchor API could evolve to support it. But even in that case, there will still be a need for anchors with arbitrary 3D poses.  Currently, anchors **are not** intended to track moving objects.
*   Persisting and sharing anchors is outside of the current scope of this explainer, since platform-level anchors (used to implement the anchor concept) are opaque and not compatible across different platforms.

Anchors are intended to maintain a pose that corresponds to a location in the physical world, and will be updated by the system as the it's understanding of the physical-world changes. 

# Use Cases

In general, applications should use anchors whenever a virtual object is placed in the scene. This is the only way to ensure the pose of the virtual object will be continuously updated to maintain a fixed relationship (i.e., position/orientation) with the physical world over time.

There are different strategies an application may use to create anchors, depending on its particular scene graph structure, but as a general use case, any virtual object that will be positioned in world space should be positioned relative to an anchor. In some cases, a single anchor might serve as the base coordinate system for multiple objects, such as when an app places a complex scene composed of multiple elements somewhere in the physical world. Only one anchor is needed in this case, as all of the elements are relative to the same phyisical location.

Imagine a small race track with cars on it. The race track will be placed at a location world space, so an anchor should be created for it, but  the cars are positioned relative to the race track, and the whole race track should move together if the anchor is repositioned. A pose update for the race track's anchor will, in turn, update the cars on top of it.

If the race track is very large, and has been laid out interactively by the user pointing at locations in the world, multiple anchors might be used for each of these locations.  In this case, an application might need to adjust the exact model of the track over time, as anchors might shift relative to each other as the user moves around the exact locations of the anchors change.  Such a scenario is more complex, but will ensure that different parts of the track are anchored to the locations the user specified, despite the system's understanding of the world evolving.

Although most use cases for anchor creation might be related to real-world understanding (e.g., placing the virtual race track on top of a real table), there are also use cases where anchors are created based on an arbitrary pose relative to the user's head (i.e., the system camera), instead of a direct relationship to the physical world. Imagine placing a virtual HUD (Heads Up Display) in mid air in front of the user to access a quick menu in AR. The placement of the UI element could be at some specific comfortable distance in front of the user, and an anchor would be needed to make sure the UI stays at the correct position/orientation even if the world coordinates change while the menu is visible.

Two examples where Anchors might update as real-world understanding improves are:

1.  The system gains a more precise understanding of where a physical object is, which affects the pose created relative to it. For example, if a pose was created relative to a plane perceived at 1m above the floor, and as the user moves around, the system refines this estimateto being 0.95m above the floor, then the Anchor pose will be updated;
1.  The system shift the world coordinate system used to specify the camera location. For example, if a pose was created (relative to the physical world, or to the camera) such that the world position is (1,1,1), and the system subsequently shifts it's world coordiantes such that the position that was formerly (1,1,1) is now (0.75,1,1), then the Anchor pose can updated accordingly.


# Possible API Considerations

*   Although anchors might be mostly created in relation to real world understanding elements (result of a hit test of a ray with the physical world, for example), there must always be a way to create an anchor at any arbitrary pose in space.
*   Because it has major implications both in terms of performance and the future requirements of concepts like sharing and persistency, anchors should be created in an asynchronous way.
*   As WebXR introduces the concept of coordinate systems in anything related to poses, and anchors are created in relation to poses, it is reasonable to assume that the anchor API should have a strong coupling to coordinate systems.
*   As the most common use case for anchors is to attach a virtual object to a location in the physical world based on the underlying system's understanding of the world, anchors might need to be tightly coupled to world understanding APIs inside the WebXR Device API. For example, an anchor should be created when placing a virtual object on the pose provided by a hit test result. Linking anchors to the hit test API proposal seems like a good idea.


# IDL proposal

```
[SecureContext, Exposed=Window] interface XRAnchor : EventTarget {
  // Attributes
  readonly attribute Float32Array modelMatrix;

  // Events
  attribute EventHandler onupdate;
};

partial interface XRSession {
  Promise<XRAnchor> addAnchor(Float32Array modelMatrix, XRCoordinateSystem coordinateSystem);
  Promise<XRAnchor> addAnchor(XRHitResult hitResult, XRCoordinateSystem coordinateSystem);
  Promise<XRAnchor> removeAnchor(XRAnchor anchor);
}
```

## API Details

`addAnchor` has 2 overloads based on the first parameter:
*   modelMatrix - a 4x4 matrix representing the initial pose where the anchor should be created. The system will make sure that the relationship with the physical world made at this moment in time is maintained as the tracking system evolves.
*   hitResult - the hit test result to create the anchor from. This overload tries to achieve 2 goals. On one hand it tries to make it easy for the developer to create anchors from a hit result, a practice expected to be common.  Hit results are expected to be a common way for users to point a places in the world, and thus a common place for developers to locate virtual content. 
*   coordinateSystem - the coordinate system the pose is relative to.
*   To enable feature detection of possible future versions of the API with  additional parameters, an error is thrown if additional arguments are given to the function.

`addAnchor` return value (applicable for both overloads)
*   Returns a Promise<XRAnchor>. The meaning of the promise resolution is as follows:
    *   If the promise rejects, there was an error (should be detailed in a returned string). It could be that the API is unsupported by the platform or the XRSession is not of the right type for creating anchors. It could be an internal error of some kind.
    *   If the promise resolves, the anchor was successfully added to the underlying system and a valid XRAnchor should be provided.
*   As this function returns a promise, the actual XRAnchor will be provided to the application in near future. In the case of anchors, the creation of the anchor should happen in the next frame and before the request animation frame of the session is called, if called during a animation frame callback.  If called outside of an animation callback, the promise might resolve before the next request animation frame, but may not. The promise should provide an XRAnchor whose internal pose (represented by the modelMatrix attribute) should always be up to date. For that reason, the app should be aware that it could be possible that the modelMatrix of the XRAnchor to be different than the original pose passed to create the anchor, and to possibly change each frame.
*   XRAnchor.modelMatrix is a 4x4 matrix representing the always up to date pose of the anchor. The app should always update any virtual objects that should be located relative to the anchor, using the value of this pose.


## Code examples

The following code examples try to clarify the proposed IDL API. They use ThreeJS to simplify some concepts representing the virtual objects/models.

### Adding anchors

```
var scale = new THREE.Vector3(); // Not really used as we are not interested in anchor pose scale
var anchorToModelMap = new Map();
// Create an arbitrary anchor
session.addAnchor(modelMatrix, eyeLevelFoR).then((anchor) => {
  // Somehow retrieve the virtual object model that will be related to the anchor.
  model.matrixWorld.fromArray(anchor.modelMatrix);
  model.matrixWorld.decompose(model.position, model.quaternion, scale);
  anchorToModelMap.put(anchor, model);
}, (error) {
  console.error(“Could not create arbitrary anchor: “ + error);
});
// Create an anchor based on a hit result
session.requestHitTest(origin, direction, eyeLevelFoR).then((hits) => {
  if (hits.length > 0) {
    session.addAnchor(hits[0], eyeLevelFoR).then((anchor) => {
      // Somehow retrieve the virtual object model that will be related to the anchor.
      model.matrixWorld.fromArray(anchor.modelMatrix);
      model.matrixWorld.decompose(model.position, model.quaternion, scale);
      anchorToModelMap.set(anchor, model);
    }, (error) {
      console.error(“Could not create the anchor based on a hit result: “ + error);
    });
  }
}, (error) {
  console.error(“requestHitTest failed: “ + error);
});
```

### Removing anchors

```
for(var anchor of anchorToModelMap.keys()) {
  session.removeAnchor(anchor);
}
anchorToModelMap.clear();
```

### Updating anchors

```
  anchor.addEventListener(“update”, (event) => {
    var anchor = event.source;
    var model = anchorToModelMap.get(anchor);
    model.matrixWorld.fromArray(anchor.modelMatrix);
    model.matrixWorld.decompose(model.position, model.quaternion, scale);
  });
```
