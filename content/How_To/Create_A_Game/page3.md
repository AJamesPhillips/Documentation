## Summary
The first step I took towards making the game was to figure out how movement would work. My past experience with 3D games pushed me towards thnking that movement would be the most difficult part of the development process, so I wanted to make sure to focus on that early on. Since I was just getting started, I knew I needed to get some prototyping in for it, so I started off by making a playground to test out simple walking, jumping, and dashing: [early prototype](https://playground.babylonjs.com/#UP84Y8#10)

[gif of movement from playground example]()

A few things you can see from this is:
1. The player is able to walk through the platform 
2. The player falls off of the platform before the mesh is "completely" off of the platform
3. And most importantly, when you jump, the player lands partially inside of the ground

This prototype underwent a lot of transformations to get to the point that it's at for the final game! For the final version, I've implemented a capsule collider + simulated rigidbody by using Babylon's collision physics and raycasting for ground detection.

[gif of movement final game]()

For part 1 we'll be going over how to detect inputs and how to get simple walking/running movement. Links to the complete files are below, but I'll be referencing certain parts that are important.

## Input Controller
For this part of the tutorial, we'll be going over the basics for movement with keyboard controls. You'll want to create a file called InputController.ts. Here we'll be creating a [PlayerInput]() class that will handle all of the inputs for our game.
```javascript
        scene.actionManager = new ActionManager(scene);

        this.inputMap = {};
        scene.actionManager.registerAction(new ExecuteCodeAction(ActionManager.OnKeyDownTrigger, (evt) => {
            if (!this._ui.gamePaused) {

                this.inputMap[evt.sourceEvent.key] = evt.sourceEvent.type == "keydown";

            } else {
                this.inputMap[evt.sourceEvent.key] = false;
            }
        }));
        scene.actionManager.registerAction(new ExecuteCodeAction(ActionManager.OnKeyUpTrigger, (evt) => {
            if (!this._ui.gamePaused) {
                this.inputMap[evt.sourceEvent.key] = evt.sourceEvent.type == "keydown";
            } else {
                this.inputMap[evt.sourceEvent.key] = false;
            }
        }));

        scene.onBeforeRenderObservable.add(() => {
            this._updateFromKeyboard();
        });
```
Within our constructor we're creating an action manager to register keydown and keyup events and using the inputMap to store whether the key was down. We're then telling the scene to call the _updateFromKeyBoard function before the scene renders.

```javascript
    if (this.inputMap["ArrowUp"]) {
        this.verticalAxis = 1;
        this.vertical = Scalar.Lerp(this.vertical, 1, 0.2);

    } else if (this.inputMap["ArrowDown"]) {
        this.vertical = Scalar.Lerp(this.vertical, -1, 0.2);
        this.verticalAxis = -1;
    } else {
        this.vertical = 0;
        this.verticalAxis = 0;
    }

    if (this.inputMap["ArrowLeft"]) {
        this.horizontal = Scalar.Lerp(this.horizontal, -1, 0.2);
        this.horizontalAxis = -1;

    } else if (this.inputMap["ArrowRight"]) {
        this.horizontal = Scalar.Lerp(this.horizontal, 1, 0.2);
        this.horizontalAxis = 1;
    }
    else {
        this.horizontal = 0;
        this.horizontalAxis = 0;
    }
```
Inside of [_updateFromKeyboard()]() we're checking for whether our arrow keys have been pressed by looking at the value that's in our inputMap. The up and down arrows are checking the vertical inputs which correspond to forward and backwards movement. The left and right arrows are checking for horizontal movement. As we press the key, we want to lerp the value so that it has a smoother transition. We are doing a couple different things here:
1. As you hold the key, it gradually increases the value to 1 or -1. 
2. We're keeping track of which axis/direction we were moving in
3. If we don't detect any inputs in an axis, we set both the direction and value to 0

## Basic Movement Setup
Now that we can detect our inputs, we need to implement what to do when those inputs are detected. We'll be focusing on the [updateFromControls]() function inside of characterController.ts.
### Input
```javascript
    this._moveDirection = Vector3.Zero(); // vector that holds movement information
    this._h = this._input.horizontal; //x-axis
    this._v = this._input.vertical; //z-axis
```
First we set up a vector3 to use as our movement vector. This will be reset every frame. Then we grab our inputs from the PlayerInput class.
```javascript
    //--MOVEMENTS BASED ON CAMERA (as it rotates)--
    let fwd = this._camRoot.forward;
    let right = this._camRoot.right;
    let correctedVertical = fwd.scaleInPlace(this._v);
    let correctedHorizontal = right.scaleInPlace(this._h);

    //movement based off of camera's view
    let move = correctedHorizontal.addInPlace(correctedVertical);
```
Now, since we want the player to move in relation to the camera, we need to grab the forward and right vectors of the camera. We then scale them by our inputs. We now have a new movement vector called move that's the combined vertical and horizontal movement. The reason why I've implemented this is because the camera view will be rotating at certain areas of the map and if the player was moving to the right as the camera rotated, we want them to be able to continue moving right even as the orientation changes. 
```javascript
    //clear y so that the character doesnt fly up, normalize for next step
    this._moveDirection = new Vector3((move).normalize().x, 0, (move).normalize().z);

    //clamp the input value so that diagonal movement isn't twice as fast
    let inputMag = Math.abs(this._h) + Math.abs(this._v);
    if (inputMag < 0) {
        this._inputAmt = 0;
    } else if (inputMag > 1) {
        this._inputAmt = 1;
    } else {
        this._inputAmt = inputMag;
    }
    //final movement that takes into consideration the inputs
    this._moveDirection = this._moveDirection.scaleInPlace(this._inputAmt * Player.PLAYER_SPEED);
```
Here, we are normalizing the vector and setting the y value to 0 since we only care about x-axis and z-axis movement. Then, we want to find the magnitude of what our combined horizontal and vertical movements give us and clamp it to be a maximum of 1 since we don't want to move faster if we're moving diagonally. We then scale our final _moveDirection by that amount multiplied by the speed we want the player to move at.

## Rotation
Now, how do we get our player to rotate towards the direction it's moving? This is where we're checking our input axes.
```javascript
    //check if there is movement to determine if rotation is needed
    let input = new Vector3(this._input.horizontalAxis, 0, this._input.verticalAxis); //along which axis is the direction
    if (input.length() == 0) {//if there's no input detected, prevent rotation and keep player in same rotation
        return;
    }
```
We first grab the input axes and check whether there were any inputs. This will determine whether or not we continue to calculate the rotation of the character. The reason why we want to return if there are no inputs is because the player will readjust their rotation to face forwards again if we don't explicitly tell it not to.
```javascript
    //rotation based on input & the camera angle
    let angle = Math.atan2(this._input.horizontalAxis, this._input.verticalAxis);
    angle += this._camRoot.rotation.y;
    let targ = Quaternion.FromEulerAngles(0, angle, 0);
    this.mesh.rotationQuaternion = Quaternion.Slerp(this.mesh.rotationQuaternion, targ, 10 * this._deltaTime);
```
Here we are calculating the angle to move the player to based off of the camera's current angle. Then we are slerping to that new target angle by a value of 10 * this._deltaTime so that we have a smooth transition as we rotate.
### Raycasts
**Raycast**

Raycasting is going to be our main method of detecting the ground beneath the character. First, we need a function [_floorRaycast]():
```javascript
    let raycastFloorPos = new Vector3(this.mesh.position.x + offsetx, this.mesh.position.y + 0.5, this.mesh.position.z + offsetz);
    let ray = new Ray(raycastFloorPos, Vector3.Up().scale(-1), raycastlen);
```
We want to send a single raycast downwards from the center of the character plus some offset if passed in, 0.5 above the bottom of the character (since the origin is at the bottom of my character mesh).
```javascript
    let predicate = function (mesh) {
        return mesh.isPickable && mesh.isEnabled();
    }
    let pick = this.scene.pickWithRay(ray, predicate);
```
Then, we want to define what can be picked by our raycast. This was important to have since I created custom collision meshes for the parts of the environment that had more complex geometry. These meshes are invisible, but should still be pickable. We start checking whether our raycast has hit anything by using pickWithRay.
```javascript
    if (pick.hit) { 
        return pick.pickedPoint;
    } else { 
        return Vector3.Zero();
    }
```
If our ray has hit anything, return the pickedPoint, a vector3. Else, we return the zero vector.

**Grounded**

The [_isGrounded]() function checks whether or not the player is on a ground by sending a raycast.
```javascript
    if (this._floorRaycast(0, 0, .6).equals(Vector3.Zero())) {
        return false;
    } else {
        return true;
    }
```
The raycast that we send is at the center of our character and extends 0.1 past the bottom. We want it to extend a little further so that it detects the ground before the player has a chance to intersect it.

**Gravity**

Now that we're able to detect the ground, we need to apply gravity to the player to keep them grounded! The [_updateGroundDetection]() will handle everything that has to do with gravity.
```javascript
    if (!this._isGrounded()) {
        this._gravity = this._gravity.addInPlace(Vector3.Up().scale(this._deltaTime * Player.GRAVITY));
        this._grounded = false;
    }
```
If we're not grounded, we want to add to our gravity and set our _grounded flag to false.
```javascript
    //limit the speed of gravity to the negative of the jump power
    if (this._gravity.y < -Player.JUMP_FORCE) {
        this._gravity.y = -Player.JUMP_FORCE;
    }
    this.mesh.moveWithCollisions(this._moveDirection.addInPlace(this._gravity));
```
Then, we want to apply gravity to our player by adding it to the current _moveDirection and moving the player by that vector.

```javascript
    if (this._isGrounded()) {
        this._gravity.y = 0;
        this._grounded = true;
        this._lastGroundPos.copyFrom(this.mesh.position);
    }
```
If the player is grounded, we want to set the gravity to 0 to keep our player grounded and set our _grounded flag to true. In addition, we'll update our _lastGroundPos to our current position to keep track of our last safe grounded position (we'll be using this later on).

# Further Reading
[Character Movement Part 2](/how_to/page4)

# Github Files
[Character Controller]()
[InputController]()