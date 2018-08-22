# What Is An Anchor?

An anchor is a concept that allows applications to specify poses, a position and orientation in three dimensional space, that will be tracked by the underlying system. There are systems where pose tracking is based on their understanding of the world. That understanding, and thus the tracking, for different reasons, varies over time. Anchors allow to specify poses that need to be updated to correctly reflect those variations to keep poses always up to date and correct.

Augmented Reality systems are examples of systems that are constantly evolving their understanding of the world, both the understanding of the user pose (via tracking a mobile device or HMD) as well as the understanding of real world objects like planar targets and faces, or semantic understanding of objects like a car or table. Anchors allow developers to specify poses to be tracked in absolute three dimensional space or relative to an identified real world object. The most important concept is that this scene understanding could evolve and thus, poses that are relevant to the application may also need to be updated and evolve over time as either the user or an identified real world object moves/rotates to always correctly represent the position and orientation where the virtual object was placed according to the real world in the first place.


# Terminology



*   **Pose**: a three dimensional position and orientation in a three dimensional coordinate system. It can be represented in many different forms but the most common ones are usually a 4x4 transformation matrix or a 3 value vector for the position and a 4 value quaternion for the orientation.
*   **Anchor**: a concept that allows developers to specify a pose that can change over time either because the whole understanding of the world coordinates (tracking) changes, or because the understanding of an object in the world the anchor is relative to changes.
    *   **Important note about the concept of anchor in Apple's iOS [ARKit](https://developer.apple.com/documentation/arkit) and in this explainer:** iOS ARKit's SDK uses the concept of an anchor to represents 2 ideas at the same time:
        1.  An arbitrary pose in 3D space that needs to be tracked. Anchors that represent an arbitrary pose can be created and registered in ARKit. This is the same as the anchor concept in this explainer.
        1.  A real world object that the system is able to identify from the real world understanding. These elements also happen to have have a pose (every element has one) but they also include other information like geometry. At the moment of the publication of this explainer ARKit is able to understand [ARPlaneAnchor](https://developer.apple.com/documentation/arkit/arplaneanchor), [ARFaceAnchor](https://developer.apple.com/documentation/arkit/arfaceanchor) and [ARImageAnchor](https://developer.apple.com/documentation/arkit/arimageanchor) as anchors.

    The main difference is that while ARKit uses the concept of an anchor to represent the identified real world object (that happens to have a pose) while this explainer uses the term anchor to only represent poses. Basic arbitrary [ARAnchor](https://developer.apple.com/documentation/arkit/aranchor) (1) in ARKit would be equivalent of the concept of an anchor in this explainer. The representation of the real world objects is out of the scope of this explainer. This differentiation between the concept of an anchor in ARKit and in the scope of this explainer is subtle but important.



# Scope

Anchors can represent different concepts:



1.  An arbitrary three dimensional pose that must be updated as the understanding of the world coordinate system evolves.
1.  A pose relative to a specific real world object the system has been able to identify and track.

Anchors could also represent poses that:



1.  Persist, meaning that the anchor is able to live between executions of the same application.
1.  Are shared between different applications.

This explainer focuses on just the concept of arbitrary anchors (1) but opening the possibility for (2) mainly for API simplicity and to allow the developer to try to always do the right thing. Creating anchors when placing virtual objects once the system has some understanding of the real world around it (hitting on a real world object) is a very common practice.

The reasons for this limited scope are:



*   Arbitrary anchors are the most basic yet useful type of anchors. There is always a need, no matter the granularity of the real world understanding the system has, to create arbitrary 3D poses as anchors. Moreover, arbitrary 3d pose anchors can easily be used to represent relative positions if eventually world understanding objects were to be exposed in the web.
*   Understanding real world objects are outside of the scope for this explainer. Currently, there is no specific proposal on how to expose real world understanding objects on the web. Whenever such proposal is agreed upon, the scope of the anchor API could evolve to support it. Even in that case, there will be still a need for arbitrary 3d pose anchors.
*   Persisting and sharing anchors is outside of the current scope of this explainer as they add much complexity and because there are still not clear technical paths on how to resolve these scenarios in the current AR platforms.

Anchors are intended to update poses as the system's real-world understanding improves - they are intended to improve precision. They **are not** intended to track moving objects.


# Use Cases

In general, applications should use anchors whenever a virtual object is placed in the scene. This is the best way to ensure the pose of the virtual object will be continuously updated to the most accurate possible pose to reflect the original real world position/orientation it represented when it was placed as the underlying tracking technology evolves.

There are different strategies an application may use to create anchors depending on its particular scene graph structure, but as a general use case, any virtual object that will be positioned in world space should have a corresponding anchor. In some cases, a single anchor might work for multiple objects, like if, for example, the app places a virtual world composed by elements inside that world. Only one anchor for the world is needed in this case as the other elements should be children of it. Think of a race track with cars on it. The race track will be placed in world space, so an anchor should be created for it, but then the cars will be placed as "children" of the race track, as they should move if the whole race track is repositioned. A pose update for the race track's anchor will, in turn, update the cars on top of it too. Thus, even though cars also are virtual objects, they do not need their own dedicated anchors in this scenario.

Although most of the use cases of anchor creation might be related to real world understanding (placing the virtual race track on top of a real table for example), there are use cases to create anchors based on arbitrary poses that do not have any direct relationship with real world objects, but with the camera pose. Imagine placing a virtual HUD (Heads Up Display) in mid air in front of the user to access a quick menu in AR. The placement of the UI element could be at some specific comfortable distance in front of the user and an anchor would be needed to make sure the UI stays at the correct position/orientation even if the tracking changes underneath.

Anchors update as real-world understanding improves. Two examples where that might occur are:



1.  The system gains a more precise understanding of where a physical object is, which affects the pose created relative to it. For example, if a pose was created relative to a plane perceived at 1m above sea level, and the system subsequently learns that the plane is actually 0.5m above sea level, then the Anchor can update the pose;
1.  The system gains a more precise understanding of where camera location was at time of when anchor was created. For example, if a pose was created relative to camera position at time t=0.5 with position (1,1,1), and the system subsequently learns that the camera was actually at position (0,1,1) at time t=0.5, then the Anchor can update the pose accordingly.


# Possible API Considerations

*   Although anchors might be mostly created in relation to real world understanding elements (result of a hit test for example), there must always be a way to create an anchor at any arbitrary pose in space.
*   Both because is a current trend in web based APIs and also because it has major implications both in performance and the future proofness of concepts like sharing and persistency, there are plenty of reasons why anchors should be created in a asynchronous way.
*   As WebXR introduces the concept of coordinate systems in anything related to poses, and anchors are created in relation to poses, it is logical to deduct that the anchor API should have a strong coupling to coordinate systems.
*   As the most common use case for anchors is to attach a virtual object to a world understood location by the underlying system, anchors might be tightly coupled to world understanding APIs inside the WebXR Device API. For example, an anchor should be created when placing a virtual object on the pose provided by a hit test result. Linking anchors to the hit test API proposal seems like a good idea.


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
*   modelMatrix - a 4x4 matrix representing the initial pose where the anchor should be created. The system will make sure that the pose relation with the real world made at this moment maintains as the tracking system evolves.
*   hitResult - the hit test result to create the anchor from. This overload tries to achieve 2 goals. On one hand it tries to make it easy for the developer to create anchors from a hit result, a very common practice as hit results are most likely used to place virtual objects in the scene. On the other hand it allows in the future to enable the anchor creation relative to natively detected trackable objects (world understanding).
*   coordinateSystem - the coordinate system the pose is relative to.
*   To enable feature detection of possible future versions of the API with  additional parameters, an error is thrown if additional arguments are given to the function.

`addAnchor` return value (applicable for both overloads)
*   Returns a Promise<XRAnchor>. The meaning of the promise resolution is as follows:
    *   If the promise rejects, there was an error (should be detailed in a returned string). It could be that the API is unsupported by the platform or the XRSession is not of the right type for AR. It could be an internal error of some kind.
    *   If the promise resolves, the anchor was successfully added to the underlying system and a valid XRAnchor should be provided.
*   As this function returns a promise, the actual XRAnchor will be provided to the application in a foreseable future. In the case of anchors, the creation of the anchor should happen in the next frame and before the request animation frame of the session is called. The promise should provide an XRAnchor which internal pose (represented by the modelMatrix attribute) should always be up to date. For that reason, the app should be aware that it could be possible that the modelMatrix of the XRAnchor has slightly changed from the original pose passed to create the anchor.
*   XRAnchor.modelMatrix is a 4x4 matrix representing the always up to date pose of the anchor. The app should always update any virtual objects that should be related to the anchor using the value of this pose.


## Code examples

The following code examples just try to clarify the proposed IDL API. They use ThreeJS to simplify some concepts representing the virtual objects/models.

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
