# Input Components and SetupPlayerInputComponent

## Main goal
- Abstract binding input actions into an independent process, that can be easily
  set up on a specific controller, character or pawn
- Is it possible to set up action bindings without specializing a Pawn class?
  - E.g. if we can dynamically add components, can we add a component that binds
    relevant input actions without having to inject code into protected methods
    of the pawn class?
- When binding input actions, minimize the dependencies on obscured state and
  control flow defined in the Pawn Class
- What is the minimum viable reduction of Lyra's HeroComponent class?

## Unanswered Questions
- **What purpose does the PlayerController's stack of input components serve?**
  - **What does it mean for an input component to be "processed" by the stack?**
  - **What does it mean for the input component to be processed by PlayerInput**
- **What does it mean for an action binding to consume an input event?**
- **What specific behavior does UInputMappingContext::Priority replicate?**

### Why does EnhancedInputComponent inherit from InputComponent?
- The player controller keeps a stack of input components, enhanced input
  component must be a subclass in order to be part of this stack

- **What purpose does the PlayerController's stack of input components serve?**

### Does the Pawn expose any sort of interface to notify when player input is set up?
- The player input component is said to be set up whenever the pawn is possessed
- There are no delegates on either pawn or controller that are broadcasted when
  possession happens

### Should we only set up input using `SetupInputComponent`?

### Does the EnhancedInputComponent make use of its owning pawn at all?
- Searching for "owner", "parent", "pawn", "character" and "controller" did not
  show any code related to the enhanced input component using information from
  the owning pawn

- This leads me to beleive that the enhanced input component does not
  _functionally_ need to be a component on the pawn. The only justification I
  can see is that the PlayerController needs the EnhancedInputComponent to be an
  InputComponent in order to "manage" it.

### Does the InputMappingContext replace the responsibility of managing input mappings?

- From the documentation for EnhancedInputComponent class:

  - An Enhanced Input Component is a transient component that enables an Actor
    to bind enhanced actions to delegate functions, or monitor those actions.

  - Input components are processed from a stack managed by the PlayerController
    and processed by the PlayerInput.

  - These bindings will not consume input events, but this behaviour can be
    replicated using UInputMappingContext::Priority.

- **What does it mean for an input component to be "processed"**
- **What does it mean for a binding to consume an input event?**
- **Does using UInputMappingContext::Priority replicate consuming input or does it replicate "stack processing" from the PlayerController?**
