---
layout: default
title: Node Types
parent: LLM StateTree
nav_order: 3
---

# Node Types

LLMStateTree supports the following StateTree node types. All node types are automatically discovered from UE reflection via the schema system.

---

## State Types

### State

Basic state that executes tasks and transitions to other states.

```json
{
  "name": "Idle",
  "type": "State",
  "tasks": [...],
  "transitions": [...]
}
```

### Group

Container state that manages child states with a selection behavior.

```json
{
  "name": "Patrol",
  "type": "Group",
  "selectionBehavior": "TrySelectChildrenInOrder",
  "children": [...]
}
```

### Selector

Utility AI selector - evaluates children and selects highest utility.

```json
{
  "name": "SelectAction",
  "type": "Selector",
  "selectionBehavior": "UtilityMax",
  "children": [...]
}
```

### Sequencer

Executes children in sequence, succeeds when all complete.

```json
{
  "name": "AttackSequence",
  "type": "Sequencer",
  "children": [...]
}
```

---

## Tasks

Tasks are behaviors executed within a state.

### StateTreeDelayTask

Wait for a specified duration before succeeding.

| Property | Type | Description |
|----------|------|-------------|
| `Duration` | float | Time to wait in seconds |
| `RandomVariance` | float | Random deviation range |

### StateTreeMoveToTask

Move to a target location.

| Property | Type | Description |
|----------|------|-------------|
| `AcceptableRadius` | float | Distance to consider arrival |

### StateTreeRunParallelStateTreeTask

Run another StateTree in parallel.

| Property | Type | Description |
|----------|------|-------------|
| `StateTree` | object | Reference to another StateTree asset |

### StateTreeDebugTextTask

Draw debug text on HUD.

| Property | Type | Description |
|----------|------|-------------|
| `Text` | string | Debug text to display |
| `TextColor` | color | Text color |
| `FontScale` | float | Font scale |

---

## Conditions

Conditions are boolean checks for transitions.

### StateTreeCompareIntCondition

Compare two integers.

| Property | Type | Description |
|----------|------|-------------|
| `Operator` | enum | Less, LessOrEqual, Equal, etc. |
| `Left` | int | Left operand |
| `Right` | int | Right operand |

### StateTreeCompareFloatCondition

Compare two floats.

| Property | Type | Description |
|----------|------|-------------|
| `Operator` | enum | Less, LessOrEqual, Equal, etc. |
| `Left` | float | Left operand |
| `Right` | float | Right operand |

### StateTreeCompareBoolCondition

Compare two booleans.

### StateTreeCompareDistanceCondition

Compare distance between two vectors.

| Property | Type | Description |
|----------|------|-------------|
| `Operator` | enum | Less, LessOrEqual, etc. |
| `Source` | vector | Source location |
| `Target` | vector | Target location |
| `Distance` | float | Distance to compare against |

### GameplayTagMatchCondition

Check if actor has a gameplay tag.

| Property | Type | Description |
|----------|------|-------------|
| `Tag` | gameplaytag | Tag to check |
| `bExactMatch` | bool | Require exact match |

---

## Considerations

Considerations are utility AI factors that produce a score.

### StateTreeConstantConsideration

Constant score.

| Property | Type | Description |
|----------|------|-------------|
| `Constant` | float | Fixed score value |

### StateTreeFloatInputConsideration

Score based on float input with response curve.

| Property | Type | Description |
|----------|------|-------------|
| `Input` | float | Raw input value |
| `Min` | float | Minimum input |
| `Max` | float | Maximum input |
| `DefaultValue` | float | Default when out of range |

---

## Transitions

Transitions define how the AI moves between states.

```json
{
  "trigger": "OnStateCompleted",
  "type": "GotoState",
  "target": "Patrol"
}
```

### Trigger Types

- `OnStateCompleted` - State finished executing
- `OnStateFailed` - State reported failure
- `OnEvent` - Custom event received

### Transition Types

- `GotoState` - Move to specified state
- `EvaluateConditions` - Evaluate conditions before transition

---

## Binding Expressions

Use `${}` syntax to reference parameters and context data:

- `${Param.Speed}` - Reference a parameter
- `${Context.Target.Location}` - Reference context property
- `${Param.IdleDuration * 2}` - Expressions supported
