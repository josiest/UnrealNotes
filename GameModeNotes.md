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
