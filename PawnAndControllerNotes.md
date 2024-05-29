# Unreal GameMode/PlayerController notes

## Initial Questions
- How is the player set up in the world?
- How is input (movement, camera, etc.) st up and where?
- How is the camera set up and how is it separate from the character object?
- What differentiates the classes PlayerController, Pawn, Character
- From GameMode's cached class data, what are minimum viable classes needed for
  a single player experience?

### What are the core GameMode classes for singleplayer?
I _think_ it is:
- DefaultPawnClass: APawn
- PlayerControllerClass: PlayerController

### What differentiates a pawn from a player controller
- in order to anaswer this, let's first understand what a pawn is used for
- A good example to see what a pawn is used for is to look at the DefaultPawn,
  the default pawn that's spawned in the default world setup when you start a
  new unreal project
- It may also be productive to compare this to Lyra's "DefaultHero" class
- Judging from research below, especially from unreal website's documentation:
  The player controller is a more permanent an abstraction from the transient
  pawn
- The controller can be a useful place to subscribe to more low level
  input events (despite how DefaultPawn is set up) since it will exist
  regardless of which pawn it controls or if the pawn "dies"
- It might be an interesting abstraction to translate low level input events
  into more semantically meaningful "movement" events that are then fed to
  the pawn that's being controlled

### How is the DefaultPawn set up?
- it overrides methods:
  - GetMovementComponent
  - SetupPlayerInputComponent
  - UpdateNavigation Relevance (is this relevant for minimum viable design?)

- defines virtual methods all related to movement that are also blueprint
  callable
  - Do these methods have a definition? If so what do they do?
  - How are these methods used?

- What happens in SetupInput?
- What components are added to the default pawn, how are they set up, and what
  are their responsibilities?

### What componets are added to the default pawn:

- Collision Component
    - set up with hard coded radius (probably want to make variable at least)
    - uses predefined collision profile with `SetCollisionProfileNmae(Pawn_..)`
    - likely responsibility is to collide w/ground and other game objects

- Movement Component
    - simple setup, only sets `UpdatedComponent` to the collision component
    - the bindings to the player input component seem to use the movement
      component indirectly through `AddMovementInput`

- Mesh Component
    - lots of settings are set up here
    - also uses pawn collision profile
    - responsibility: root component for visuals

### What happens in SetupInput?
- Manually adds hard-coded mappings:
    `DefaultPawn_Move{Direction}` -> `Ekeys::{Input Key/Button}`
                                     and 1-dimensional direction
- Should this be done with EnhancedInput?
- What examples are there that show how to set this up w/enhanced input (does
  Lyra have any such examples?)
- Mappings are first added to the engine, but then mappings that _aren't_
  hard-coded (but still defined as static member variables) are added to the
  player input component
   - why are there two places where input is set up?
   - what is an `EngineDefinedAxisMapping`?
     - it may be easiest to answer this question by creating a custom default
       pawn and testing first with mappings added then by taking away
   - does it _need_ hard-coded values?
   - can we use loaded input actions at this point?
   - when is SetupPlayerInputComponent called? Is it before the enhance input
     system is initialized?
   - when is the enhanced input system initialized?

### When is SetupPlayerInputComponent called?

- SetupInputComponent is called indirectly whenever a pawn is possessed
- **When are pawns possessed?**
    - Seems in a few different places:
    - When AI controllers are spawned
    - In the pawn's `PreInitializeComponents()`, the pawn possesses the PC
    - In a level, when "auto receive input"
    - From the game mode base in `FinishRestartPlayer` the new player
      controller possesses its pawn
- **When is PreinitializeComponents called**?
  After the actor is spawned, but before components are initialized
- **When is FinsishRestartPlayer called and is it relevant to single player?**
  - Called whenever a "match" has started, but isn't a "match" just the way
    unreal says "existing in/loading a level?"
  - GameMode seems to use a state machine called a "match"
  - InitGame calls `SetMatchState(EnteringMap)` but there are other states that
    are set at different points: (e.g. InProgress, LeavingMap, etc.)

### What is the responsibility of the Controller?
-  Unreal Documentation for `AController`:
  - Controllers are non-physical actors that can possess a pawn to control its
    actions. PlayerControllers are used by human players to control pawns, while
    AIContollers implement the artificial intelligence for the pawns they
    control. Controllers take control of a pawn using theri Possess() method,
    and relinquesh control of the pawn by calling UnPossess()

    Controllers receive notifications for many of the events ocurring for the
    Pawn they are controlling. This gives teh controller the opportunity to
    implement the behavior in response to this event, intercpeting the event and
    superceding the Pawn's default behavior.

    ControlRotation (accessed via GetControlRotation()) dtermines the
    viewing/aiming of the controlled pawn and is affected by input such as from
    a mouse or a gamepad

  - Keeps a reference to "PlayerState" and "StartSpot" where it spawned. Also
    has a "StateName" member. Does this keep track of a current state in some
    state machine?

  - Private members: keeps track of its controlled pawn, and "old" controlled
    pawn for changing pawns (is this needed for single player?). Finally, it
    also keeps track of the character being controlled. **If a character is
    controlled the player controller, do `Pawn` and `Character` refer to the
    same object? **

  - Most of Controller's code seems to be related to multiplayer? Although
    there is some "rotation" related code which seems a little weird that this
    is specifically separated from the rest of the movement code that's set up
    in Pawn implementations

### What responsibility does PlayerController add to the Controller class?
- From the unreal documentation:
  - In networked games, PlayerControllers exist on the server for every
    player-ctonrolled pawn, and also on the controlling client's machine. They
    do NOT exist on a client's machine for pawns controled by remote players
    elsewhere on the network.

    One thing to consider when setting up your PlayerController is what
    functionality should be in the PlayerController, and what should be in your
    **Pawn**. It is possible to handle all input in the Pawn, especially for
    less complex cases. However, if you have more complex needs, like multiple
    players on one game cient, or the ability to change characters dynamically
    at runtime, it might be better to handle input in the PlayerController. In
    this case, the PlayerController decides what to do and then issues the
    commands to the Pawn

    Also, in some cases, putting input handling or other functionality into the
    PlayerController is necessary. The PlayerController persists throughout the
    game, while the Pawn can be transient. For example, in deathmatch style
    gameplay, you may die and respawn. So you would get a new Pawn, but your
    PlayerController would be the same. In this example, if you kept your score
    on your Pawn, the score would reset, but if you kept your score on your
    PlayerController, it would not.
