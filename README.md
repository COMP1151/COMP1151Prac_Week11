# Week 11 -- Physics & Rigidbodies --

Learning outcomes:

* Become familiar with the different types of forces in a physics engine (velocity, impulse)
* Understand the impact of "Gravity" in games
* Familiarisation with different (physics based) control schemes, and their purposes
* Able to create and tweak physics based interactions between a player controlled character, and stationary objects in world

## Foreword

Today we are going to be working in the same repository from a few weeks ago. However, we will not be working in the same scene - we will be learning about aspects of the Unity Physics system, and this system is far easier to explore in a different topography (more akin to a platformer)

Also, the content here is meant to be fairly light. We are being conscious of how much time this prac should take, and how much key content it should contain (that is relevant to your assignment). As a result, this is simply a cursory overview of the physics system; physics will be covered in greater detail in your second year subjects (or you can look into it yourself)

One thing to keep in mind using physics though, a physics system in of itself is a shortcut to manually coded interactions - to quote the lecture slides:

" Use the physics system sparingly and knowledgeably.

It will often get you 90% of the way to a solution, then make the final 10%
extremely difficult."

## Setup

To get started, we require a new level layout, among other things:

1. Make a new **Scene** in your Unity project, called "`Physics`"
2. Using **Tiled**, make a new **TileMap** (`.tmx` file). This tilemap should simply be a large box the player can walk around in, made with wall colliders (image below)
   a. Once completed, place this new Tilemap into your `Physics` **Scene**
   b. Ensure this has a `Tilemap Collider 2D` object attached to its layer - check your other scenes implementation if you are not sure where to place this
3. A new player prefab, with a new `Script Graph` - Make sure they have an easily identifiable name, like "`Player_Physics`"
   a. It is recommended to simply duplicate (Select the file/prefab in your Project window, and right-click > `Duplicate`)
   b. The new `Script Graph` should be a duplicate of your old Player script graph
   c. Alternatively, you can "unpack" your existing Player prefab, and make the changes from there. Ask your TA to show you how if you are curious about this method
   d. It may help if you give `Player_Physics` a different sprite, so you don't get confused

You are ready to continue when you have something like this:

<img src="PracResources\images\00_SceneSetup.png">
<img src="PracResources\images\00_2_GraphSetup.png">

## Gravity

If you followed the steps correctly from the previous week, you should already have a `Rigidbody2D` component on your player. However, we made some changes in the previous prac without really knowing what they do, so lets go back over those changes and get a better understanding of them

Before moving on, if you don't have a `Rigidbody2D` on your player already, put one on now. Otherwise, reset your `Rigidbody2D` by right clicking the component in the **Inspector**, and selecting `Reset`

<img src="PracResources\images\01_RBReset.png">

As you might guess, this resets the component state to how it was initially implemented, and can be a good trick to remember when troubleshooting bugs

Let's get started, we're going to make a simple test to ensure our gravity is working properly. Place the player in the above the floor of your box like so:

<img src="PracResources\images\02_BoxTestInitial.png">

Press Play. Your player should slowly be sliding downwards, but this is at an unnatural rate. This is because we actually already have something affecting our physics, our movement implementation from last week.

Go into your Player_Physics script graph, and delete the `Linear Velocity2D` node. Then replay your game, and your player should fall like you would expect (it should start slow, but increase in its downwards velocity over time).

### Tweaking Gravity

This is neat, but what if we want to change the speed that we fall down. You might think that the `Mass` field in the players `Rigidbody2D` component is the way to go about this. Give this a try - Make a small experiment by making a duplicate of the player object, one with `Mass: 10` and the original with `Mass: 1`, and notice the difference in their fall speed.

<img src="PracResources\images\03_MassExperiment.png">

Mathematicians in the class might immediately already know what's happening, but essentially:

```
*Mass* does not impact falling speed.
```

This can be an easy thing to forget when making games in Unity, so it's worth doing this exercise to confirm this yourself. Instead, `Mass` in Unity affects the collision system - Higher mass objects apply more force to the other object when pushing them around


|   |                                                                                                                                  For example...                                                                                                                                  |   |
| --- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | --- |
|   | Imagine a big truck ramming a beach ball - you would imagine the beach ball would go flying, and the truck would essentially keep all of its force - if Mass is set up correctly for both objects, this interaction should occur as expected (from a common sense point of view) |   |
|   |                                                                                                                                                                                                                                                                                 |   |

Instead, the following affect falling speed (using Gravity):

* `Gravity Scale`: The degree to which an object is affected by gravity
  * Changing a birds Gravity scale, for example, could make more sense than other implementations
  * It is not advised to give every object a unique Gravity Scale, and instead this should be normalised
* `Gravity` (@ `Edit > Project Settings > Physics 2D`): Changes the gravity for **the entire project**.
  * Universal changes to gravity should be made here
  * By default, this value is `Y: -9.81`
  * By default this changes gravity for the entire project, but a script can be made (that affects this) to change this on a by-level basis
* `Linear Damping`: Used to reduce the magnitude (impact) of linear *speed*. Zero indicates no damping should be used
  * Akin to air drag
  * An implementation of deploying a parachute could be used using `Linear Damping` - If the player were to deploy a parachute, you could simply implement this in code by increasing the `Linear Damping` while the parachute is opening (and reverting the linear damping when it is closed)
* `Angular Damping`: Affects the magnitude (impact) of linear *rotation*.
  * It is unlikely you will need to adjust this in most cases
  * An example of an implementation of this could be forcibly slowing down a spinner/carousel at a playground

Perform the following experiments to learn more about these:

* Change the `Linear Damping` of your two player objects, give the one with `Mass: 10` a `Linear Damping` of `0`, and the player with `Mass: 1` a `Linear Damping` of `1`, and observe the difference
* Revert the changes to `Linear Damping`, but then change the `Gravity Scale` for one of the players. Again, observe the difference
* Turn off Gravity entirely for one of the players (i.e. set their `Gravity Scale: 0`). Observe what happens
* Change the impact of Gravity for the whole scene (in `Edit > Project Settings > Physics 2D`). Observe the difference in speed that the player falls

## Moving the player with Physics

Let's change pace a bit. Delete your second player object, and change the `Mass`/`Gravity Scale`/`Linear Drag` on your original player, and the global `Gravity` back to normal.

Open your `Player_Physics` script back up. We're going to look at the different types of movement we can use when using Physics.

Let's start with what we had originally, `Linear Velocity`. However, we can't really use this in our case, since we don't want it to affect our gravity


|   | Experiment                                                                                                                                                                                     |   |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --- |
|   | You can see what happens when this is implemented with gravity if you would like, by simply adding the`Linear Velocity` node back in. However, you will see that it isn't really usable for us |   |
|   |                                                                                                                                                                                                |   |

### Linear Velocity

Thankfully, there's an alternative to this, simply using a `Set Linear Velocity X` node:

<img src="PracResources\images\10_LinearVelocityX.png">

Have a go at implementing this yourself, but keep in mind the following:

* Do not connect `On Update` to `Set Linear Velocity`. Instead, delete `OnUpdate` entirely
* From your multiply node, you will need to extract the X value *only*, using a `Vector2: Get X` node, if you simply pass in a Vector2 in to this node, it will not work
* Ensure that your `OnInput` node it set to `On Pressed`

Play your game, and move left/right. You will notice that this movement feels quite odd, as if it is skating on ice. There is one issue with this implementation though, we are currently using `On Pressed` - this means that we are essentially shoving the player, as opposed to having them continuously move while we are holding our desired movement. Change the following:

* Set your `OnInput` node to `On Hold`, as opposed to `On Pressed`

<img src="PracResources\images\11_ChangeToOnHold.png">

Again, this movement is quite strange, it is if our player is moving on ice while we are holding left/right down. While this isn't really useful for implementing Player movement, you could imagine how this could be used in a gameplay event, or as an outcome of a physics interaction (e.g. kicking an ice hockey puck, or a soccer ball)

Let's implement this properly:

* Create a `FixedUpdate` node, and hook it up to `Set Linear Velocity X`
* Set your `OnInput` node back to `On Pressed`

<img src="PracResources\images\12_RevertOnHold.png">

This is about what you'd expect for movement, but coming from the above example, it feels a bit stiff/jerky. However, this implementation isn't ready for interaction, as simply setting the velocity leads to unrealistic physics simulations for the majority of cases; we should use something else.


|   | Why FixedUpdate                                                                                                                                                                                                                                             |   |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
|   | Students who watched last weeks lecture already know why we are using OnFixedUpdate for here (as opposed to simply just OnUpdate). If you are unsure why we do this, it is recommended to watch last weeks lecture, as the information is fairly important. |   |
|   |                                                                                                                                                                                                                                                             |   |

### AddForce - Force

Let's look at that "something else" now - adding force. Specifically the `Add Force X` node. Looking at the node, we can see that this immediately looks a bit different to our Linear Velocity Node:

<img src="PracResources\images\13_AddForceNode.png">

The obvious difference is the **Mode** dropdown. When using `Rigidbody2D`, we can set **AddForce** to two different modes:

* `Mode: Force`: This adds a force to the Rigidbody2D, using its mass
  * This is useful for setting up realistic physics, such as where it takes more 'force' to move heavier objects (using our analogy from before, our truck trying to tow something stuck in the mud)
* `Mode: Impulse`: Adds an *instant* force impulse to the Rigidbody2D, using its mass
  * This is useful for instantly occurring forces, such as forces resulting from explosions or collisions (again, think back to the Truck ramming the beach ball)

Replace the `Set Linear Velocity X` node with the `Add Force X` node, ensure it is set to `Mode: Force`, and press Play


|   |                                                                                                                                                                                                   Why does my character spin like a soccer ball?                                                                                                                                                                                                   |   |
| --- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | --- |
|   | Depending on the type of**Collider** you have placed on your player, your Player will immediately start spinning like a ball once you play your game. Take a moment to think about it, and this would technically make sense (computers do what they tell us to after all). This is a fairly common issue, and the fix is quite simply - In your Players`Rigidbody2D`, extend the `Constraints` dropdown, and check the `Freeze Rotation: ✅Z` box. |   |
|   |                                                                                                                                                                                                                                                                                                                                                                                                                                                     |   |

<img src="PracResources\images\xx_FreezeRotationZ.png">

As mentioned before, this movement is impacted by the  **Mass** of the `Rigidbody2D`, and `Linear Damping` is essentially acting like the objects drag.

Ensure you have exited **Play** mode (Remember Rigidbody2D does not save when in Play mode!), and then tweak these three variables until you have a movement set up you are happy with:

* `Mass`
* `Linear Damping`
* `movementSpeed` (on the player Script Graph)

### AddForce - Impulse

While we're tweaking things, lets take a brief look at **Impulse**. As mentioned before it is generally used for applying *instant* force, so with our current set up, it will be hard to critically assess its impact.

Sharp students will already know what we are going to change here:

* Ensure our `OnInput` node back to `OnPressed`
* Disconnect our `OnFixedUpdate` node (we will only want this to happen once per key press)
* Change our `Force Mode X` node to Mode: `Impulse`

<img src="PracResources\images\14_ImpulseMode.png">

Again, play around with the `Mass`, `Linear Damping`, and `movementSpeed`. See if you can create something akin to a "dash" movement, by setting a low mass, high linear damping, and the same movement speed as normal.

If you've come up with something you prefer over your old movement system, you're more than welcome to keep it. However, it is recommended to go back to `AddForce(Force)` before moving on.

## Physics Materials

We're now going to look at how different physics interactions can affect our player, using the physics system itself. Go back into **Tiled**, and make a new Tile layer in our **Tilemap** called "BouncyWall". We're essentially going to make a bouncy wall - like a jumping castle:

* Select your original layer, and erase the walls of your box
* Select your "BouncyWall" layer, and draw your new walls using a unique sprite (make sure this sprite has a collider!)
  * If you'd like, you can also make the area where the player drops down bouncy as well

Something like this is reasonable:

<img src="PracResources\images\20_BouncyCastle.png">

Back in Unity, make a new folder in your `Assets` folder called PhysicsMaterials, and inside it, create a new `PhysicsMaterial2D` - this can be found by right-clicking in the **Project** window, and selecting `Create > 2D > PhysicsMaterial2D`. Give it an appropriate name, like "`Bouncy`"


|   | Don't get tricked by 3D Physics                                                                                                                                                                                                                                                             |   |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
|   | It is quite hard to distinguish between 2D and 3D Physics Materials, as they look quite similar. If you run into issues where it is seems like it is not working, ensure that it is a PhysicsMaterial2D (you can verify this in the**Inspector** - the type should appear next to its name) |   |
|   |                                                                                                                                                                                                                                                                                             |   |

In the **Project** window, select the **PhysicsMaterial2D** we just made. You will see that the variables here are quite intuitive - note that both friction and bounciness are actually percentages (from 0 - 1). Give the variables values something similar to the following:

* `Friction: 0.1`
* `Bounciness: 1`
* `Friction Combine: Mean`
* `Bounce Combine: Maximum`

<img src="PracResources\images\21_BouncyMaterial.png">

In your **Hierarchy**, navigate to your `BouncyWall` object, (it should be a child of `Grid`), and give it a `TilemapCollider2D` and a `Rigidbody2D`. You might think its strange that we give a Rigidbody2D to the wall, but this is required to add our `PhysicsMaterial2D`. Change the following:

1. In Rigidbody2D, set `BodyType` to `Static`
2. Press the object picker button next to `Material`, and select our `Bouncy` Material

* If you do not see it, it is because you've made a PhysicsMaterial, **not** a PhysicsMaterial**2D**

Do this, and have a few goes ramming your player into the wall, and watch it go flying across your box. Try tweaking the variables so that there is less bounciness, and watch the difference

|   | Why isn't PhysicsMaterial my updating?                                                                                                                                         |   |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
|   | PhysicsMaterial values can not change during runtime. As a result, to test different variables for a PhysicsMaterial, you need to exit Play mode, and then re-enter Play Mode |   |
|   |                                                                                                                                                                               |   |

Let's make our other other Tilemap layer a bit more interesting:
1. Make a new PhysicsMaterial called `Slippery`
2. Give it high friction, and change the `Friction Combine` to Maximum
3. Apply this to the other Tilemap layer, the the same way you applied the BouncyWall

Play your game and run across the wall, and you will notice that... nothing changes.

This is due to a fundamental key note with how **Physics Materials** work, they still need a **collision** for anything to happen. This is why our bouncy example does work, but this solution doesn't quite work. 

If you're curious on how you *could* go about implementing an icy tilemap, pick your TA's brain, or do some research online. There are a few ways you could implement this.

## Creating your own Physics Interaction to Objects

Finally, let's start using physics systems like they're supposed to be used, by smashing into things. This task is pretty open-ended, so the end result is up to you. Implement a physics based objects with the following constraints:
* The object should use a `Rigidbody2D`, and a `Collider2D` (ideally a `Capsule` or `Circle` Collider)
* It should be small - ideally it should use a sprite from your tilesheet used to make your scene
* It should be light
* You want the player to ram it, and lose speed a small amount of speed
  * The speed loss should be *minor*, and only noticeable upon close inspection
  * The result of the impact should be that the object is launched, but the player keeps moving/is grounded
* You may (or may not) want to tweak the `Linear Drag `of the object, to make it feel realistic
* You may (or may not) want to change its `Angular Drag`, to control how much it spins when collided with
* If the object is light/floaty (like our truck/beach ball example earlier), also consider changing its `Gravity Scale`

Have a go at implementing this yourself. Once you've made one object, save it as a prefab, **duplicate them several times** and have fun!

### Show your demonstrator

When you're done, you may wish to show your player ramming into your several physics objects; show your TA just how many frames Unity loses!

# Completed Tasks:

To verify that you've completed all of the content in this prac, check you have done the following:

* Created a unique scene to test physics in
* Implemented realistic-ish gravity and drag for your Player
* Implemented Physics based movement, tweaked to your liking
* Made a "Bouncy" wall, that the player can fling themselves from
* Created a physics based object (from scratch), that goes flying when the player rams into it

## Further Activities

There are two challenges for this week. Note:
* Both of these challenges are **optional**
* You may continue on with your assignment instead if you wish

### Optional Challenge: Without Gravity/Damping, Make the Player Slow Down when mo Key Is Pressed

As the title of the challenge says, experiment with

### Optional Challenge: Implement Physics-based Movement and Interactables into your "Game" Scene

As the title says, implement both the player movement and your physics object into the scene from last week. This will take some critical thinking, as well as some tweaking of the player movement in order to get this working nicely.

* It is recommended to make these changes in the `Player` prefab, not the `Player_Physics` prefab. You may wish to rename them so the change is obvious (e.g. `Player` and `Player_TestScene`)