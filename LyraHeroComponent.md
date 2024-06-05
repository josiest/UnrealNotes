# Lyra Hero Component Notes

## Main Goals
- How does Lyra's input component set up/bind player input as a pawn component?
- What are the minimum viable features I need for a custom component with
  similar functionality?

## Basic Notes

- A PawnComponent that implements GameFrameworkInitStateInterface
  - **What benefits does the PawnComponent inheritance give?**
  - **How is the GameFrameworkInitStateInterface used?**

- Component has "default mappings and priority" data member
  - How is this used?
  - Are there other mappings that _should_ be used instead of the default?

- Lyra Hero Component sets up input _and_ camera - why combine both these
  responsibilities into one?

## How is input set up as a pawn component?
- A similar method to SetupPlayerInputComponent: InitializePlayerInput
- There are more external dependencies to find:
  - Need to get the owning Pawn
  - Need to get Pawn Extension Component from the owning pawn
  - Pawn Extension Component contains "PawnData" which contains "input config"
  - Input config is the tagged input action bindings

- uses LyraInputComponent which "has some additional fucntions to map Gameplay
  Tags to an Input Action."

- pseudo code:
  - clear all current mappings in subsystem
  - for each default mapping and priority
    - register mapping with settings if specified
    - add default mapping context
  - add the input mappings from the pawn data
  - bind each action (hard coded which actions) - e.g. move, look, etc.

- **What does it mean to register a mapping with settings?**

## How is the GameFrameworkInitStateInterface used?
- Seems to be some sort of state machine that's used in tandem with
  LyraPlayerState
- There are InitState tags that all seem to be related to different stages of
  setting up data
- **What state may be uninitialized, that this state machine helps set up?**
- **What state do the core features I need depend on,
    that may be uninitialized?**
