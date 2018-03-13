# Anchors
## Background
Immersive systems are able to track positions and orientations (also called poses) with high accuracy. Some immersive systems use tracking mechanisms (like [Visual Odometry](https://en.wikipedia.org/wiki/Visual_odometry), VO) that evolve as they get new data over the course of a session to better estimate poses. In such systems, when an application places a virtual object in a specific pose, it is important to update the pose when the evolution of the system tracking would alter the estimate of it relative to the world. Anchors provide such mechanism where the app is able to tell the underlying system that a pose should be tracked and up to date at all times. Anchors are most important in Augmented Reality systems/applications, especially in scenarios where the user moves long distances while the tracking system is still on.

## Use Cases
In general, applications should use anchors whenever a virtual element is placed in the scene. This is the best way to ensure the pose of the virtual object will be continuously updated to the most accurate possible pose from the underlying tracking technology.

There are different strategies an application may decide to create anchors depending on its particular scene graph structure, but, as a general use case, any virtual object that will be positioned in world space should have a corresponding anchor. In some cases, a single anchor might work for multiple objects, like if, for example, the app places a virtual world composed by elements inside that world. Only one anchor for the world is needed in this case as the other elements should be children of it. Think of a race track with cars on it. The race track will be placed in world space, so an anchor should be created for it, but then the cars will be placed as “children” of the race track, as they should move if the whole race track is repositioned. A pose update for the race track’s anchor will, in turn, update the cars on top of it too. Thus, even though cars also are virtual objects, they do not need their own dedicated anchors in this scenario.

Although it is recommended to use anchors everytime a virtual object is placed in a scene, some more specific scenarios can be narrowed down:

* Position a virtual object at an arbitrary point in the world (relative to the device pose): The virtual object's pose should be updated as the world understanding evolves.

* Position a virtual object on a real-world surface: The virtual object's pose should be updated as the estimate of tracking of the real world element it is attached to evolves.

## Proposed Approach
In order to provide basic anchor support, there should be a way for the app to tell the underlying system that a specific pose (e.g. in the form of a 4x4 matrix for example, a very common way to express all kinds of linear transformations) should be tracked. Anchor creation should be an asynchronous process as the underlying system could reject the creation of the anchor for different reasons. Anchors should also be created in relation to a coordinate system as its frame of reference. Once the anchor is created, there should be a way to remove it too. There should also be a way for an application to be able to listen to anchor update events, where the app is notified when the anchor’s pose (the 4x4 matrix) has been updated by the underlying system for any reason, providing enough information so that the app can update the pose of virtual objects. It is left up to the application to properly update the object's pose in response to these events.


