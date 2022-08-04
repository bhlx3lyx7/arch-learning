# UML Learning

## Structural diagrams

### Class diagram
- Generation: child -----▷ parent
- Realization: class - - -▷ interface
- Association: owner -----> thing
- Dependency: user - - -> thing
- Aggregation: tire -----⬦ car
- Composition: wing -----⬥ bird

### Component diagram
- components
- relationship: a call b for what

### Deployment diagram
- nodes
- connection

### Object diagram
instance of class diagram

### Package diagram
- use: a depend b
- merge: a include b
- access: a private import b
- import: a public import b

### Composite structure diagram
components in a whole composition
- components
- relationship

### Profile diagram
show the profiles of a domain
- concepts
- relationship

## Behavioral diagrams

### Use case diagram
- node
	+ user
	+ use case
- boundary
- relationship
	+ include: action a depend action b
	+ extend: action b extend from action a
	+ generation: child action inherit parent action

### Activity diagram
workflow of use case
- action
- flow & branch

### State machine diagram
lifecycle of one object
- state
- event & state transfer

### Sequence diagram
cooperation among components, in time sequence
- components
- trigger event with sequence
- time range

### Communication diagram
communication between components of different workflow, without time sequence
- components
- trigger event

### Interaction overview diagram
similar to activity diagram, but the node is interaction; for control workflow
- node: interaction diagram
- workflow: control process

### Timing diagram
as time advance, the state or value changes
- time axis
- state/value

## References
- 14 UML diagrams: https://cloud.tencent.com/developer/article/1684161
