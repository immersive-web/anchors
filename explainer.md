# What Is An Anchor?

An anchor is a concept that allows applications to specify poses, a position and orientation in three dimensional space, that will be tracked by the underlying system. There are systems where pose tracking is based on their understanding of the world. That tracking, for different reasons, varies over time. Anchors allow to specify poses that need to be updated to correctly reflect those variations to keep poses always up to date and correct.

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

This explainer focuses on just the concept of arbitrary anchors as stated in 1) with no relationship to any real world objects, nor any sharing or persistency capabilities.

The reasons for this limited scope are:



*   Arbitrary anchors are the most basic yet useful type of anchors. There is always a need, no matter the granularity of the real world understanding the system has, to create arbitrary 3D poses as anchors. Moreover, arbitrary 3d pose anchors can easily be used to represent relative positions if eventually world understanding objects were to be exposed in the web.
*   Understanding real world objects are outside of the scope for this explainer. Currently, there is no specific proposal on how to expose real world understanding objects on the web. Whenever such proposal is agreed upon, the scope of the anchor API could evolve to support it. Even in that case, there will be still a need for arbitrary 3d pose anchors.
*   Persisting and sharing anchors is outside of the current scope of this explainer, as they add much complexity and because there are still not clear technical paths on how to resolve these scenarios in the current AR platforms.

Anchors are intended to update poses as the system's real-world understanding improves - they are intended to improve precision. They **are not** intended to track moving objects.


# Use Cases

In general, applications should use anchors whenever a virtual object is placed in the scene. This is the best way to ensure the pose of the virtual object will be continuously updated to the most accurate possible pose to reflect the original real world position/orientation it represented when it was placed as the underlying tracking technology evolves.

There are different strategies an application may use to create anchors depending on its particular scene graph structure, but as a general use case, any virtual object that will be positioned in world space should have a corresponding anchor. In some cases, a single anchor might work for multiple objects, like if, for example, the app places a virtual world composed by elements inside that world. Only one anchor for the world is needed in this case as the other elements should be children of it. Think of a race track with cars on it. The race track will be placed in world space, so an anchor should be created for it, but then the cars will be placed as "children" of the race track, as they should move if the whole race track is repositioned. A pose update for the race track's anchor will, in turn, update the cars on top of it too. Thus, even though cars also are virtual objects, they do not need their own dedicated anchors in this scenario.

Anchors update as real-world understanding improves. Two examples where that might occur are:



1.  The system gains a more precise understanding of where a physical object is, which affects the pose created relative to it. For example, if a pose was created relative to a plane perceived at 1m above sea level, and the system subsequently learns that the plane is actually 0.5m above sea level, then the Anchor can update the pose;
1.  The system gains a more precise understanding of where camera location was at time of when anchor was created. For example, if a pose was created relative to camera position at time t=0.5 with position (1,1,1), and the system subsequently learns that the camera was actually at position (0,1,1) at time t=0.5, then the Anchor can update the pose accordingly.


# Possible API Considerations

Although this explainer tries to limit the scope of this initial take on anchors and does not try to provide a possible IDL on how the API could look like, it might still be important to take some concepts into consideration.



*   Both because is a current trend in web based APIs and also because it has major implications both in performance and the future proofness of concepts like sharing and persistency, there are plenty of reasons why anchors should be created in a asynchronous way.
*   As WebXR introduces the concept of coordinate systems in anything related to poses, and anchors are created in relation to poses, it is logical to deduct that the anchor API should have a strong coupling to coordinate systems.


# Open Questions



*   Should anchor creation be used in VR?
    *   Anchors could never update in VR scenarios but they provide positive features to the overall WebXR spec. In one hand, they may get developers to be aware of the importance of using anchors in general while developing for XR. They may provide more functionality when sharing and persistency are also proposed in the spec.
*   Could anchors represent poses that change their relationship with the understanding of the world over time?
    *   For example a helicopter landing on a platform, it may not need to have to be represented by an anchor as the platform could have been represented by one. But when the helicopter starts to fly, it is no longer influenced by the platform so it could need to be represented by an anchor to correctly reflect its pose changes. Then the helicopter could land in a different platform and be attached to it and no longer require an anchor to represent its pose.
