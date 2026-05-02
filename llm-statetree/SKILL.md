---
name: llm-statetree
description: |
  LLMStateTree is a DSL-driven UE5 StateTree asset generation system. Use this skill when users need to:
  (1) Create or design UE5 StateTree AI behavior trees (idle, patrol, chase, attack, etc.)
  (2) Convert AI behavior descriptions into generatable DSL format (.llmstate JSON)
  (3) Understand or modify existing StateTree DSL definitions
  (4) Learn about supported node types (State, Group, Selector, Sequencer, Tasks, Conditions, Considerations)
  (5) Query binding expression syntax and context data references
---

# LLM StateTree - DSL Schema Guide

### **Composition Over Inheritance**

## File Format

StateTree definition files use the `.llmstate` extension (LLM StateTree DSL), formatted as JSON.

## Reference Files

**Examples Directory**: `Plugins/LLMStateTree/Config/Examples/` - Contains complete AI behavior example `.llmstate` files

**Samples**:
- `SimpleAI.llmstate` - Basic AI with Idle, Patrol, Chase, Attack states
- `GuardAI.llmstate` - Guard NPC with patrol, investigate, chase, return home
- `UtilityAI.llmstate` - Utility AI with considerations for action selection
- `StealthAI.llmstate` - Stealth AI with hide, sneak, detect, escape states
- `BossAI.llmstate` - Boss AI with multiple phases based on health thresholds

## Quick Start

```json
{
  "name": "MyAI",
  "parameters": [
    { "name": "PatrolRadius", "type": "float", "defaultValue": 500.0 },
    { "name": "ChaseSpeed", "type": "float", "defaultValue": 600.0 }
  ],
  "states": [
    {
      "name": "Idle",
      "type": "State",
      "tasks": [
        { "type": "StateTreeDelayTask", "properties": { "Duration": 2.0 } }
      ],
      "transitions": [
        { "trigger": "OnStateCompleted", "type": "GotoState", "target": "Patrol" }
      ]
    },
    {
      "name": "Patrol",
      "type": "State",
      "tasks": [
        { "type": "StateTreeMoveToTask", "properties": { "AcceptableRadius": 100.0 } }
      ],
      "transitions": [
        { "trigger": "OnStateCompleted", "type": "GotoState", "target": "Idle" }
      ]
    }
  ]
}
```

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

## Binding Expressions

Use `${}` syntax to reference parameters and context data:

- `${Param.Speed}` - Reference a parameter
- `${Context.Target.Location}` - Reference context property
- `${Param.IdleDuration * 2}` - Expressions supported

## Parameters

Define parameters that can be set when assigning the StateTree to an AI character:

```json
{
  "parameters": [
    { "name": "PatrolRadius", "type": "float", "defaultValue": 500.0 },
    { "name": "ChaseSpeed", "type": "float", "defaultValue": 600.0 },
    { "name": "AlertRadius", "type": "float", "defaultValue": 1000.0 }
  ]
}
```

**Supported Types**: `float`, `int`, `bool`, `vector`, `gameplaytag`

## Selection Behaviors

### Group Selection Behaviors

| Behavior | Description |
|----------|-------------|
| `TrySelectChildrenInOrder` | Evaluate children in order, select first eligible |
| `TrySelectChildrenInRandomOrder` | Evaluate children in random order |
| `SelectMostDesirableChild` | Select child with highest priority |

### Selector (Utility AI) Selection Behaviors

| Behavior | Description |
|----------|-------------|
| `UtilityMax` | Select child with highest utility score |
| `WeightedRandom` | Select child based on weighted random |

## Common Patterns

### Simple Patrol Loop
```json
{
  "name": "SimplePatrol",
  "parameters": [
    { "name": "PatrolRadius", "type": "float", "defaultValue": 500.0 }
  ],
  "states": [
    {
      "name": "Idle",
      "type": "State",
      "tasks": [
        { "type": "StateTreeDelayTask", "properties": { "Duration": 2.0 } }
      ],
      "transitions": [
        { "trigger": "OnStateCompleted", "type": "GotoState", "target": "Patrol" }
      ]
    },
    {
      "name": "Patrol",
      "type": "State",
      "tasks": [
        { "type": "StateTreeMoveToTask", "properties": { "AcceptableRadius": 100.0 } }
      ],
      "transitions": [
        { "trigger": "OnStateCompleted", "type": "GotoState", "target": "Idle" }
      ]
    }
  ]
}
```

### Utility AI with Considerations
```json
{
  "name": "UtilityAI",
  "parameters": [
    { "name": "MaxHealth", "type": "float", "defaultValue": 100.0 },
    { "name": "LowHealthThreshold", "type": "float", "defaultValue": 30.0 }
  ],
  "states": [
    {
      "name": "SelectAction",
      "type": "Selector",
      "selectionBehavior": "UtilityMax",
      "children": [
        {
          "name": "Flee",
          "type": "State",
          "considerations": [
            {
              "type": "StateTreeFloatInputConsideration",
              "properties": {
                "Input": "${Param.Health}",
                "Min": 0.0,
                "Max": "${Param.MaxHealth}",
                "DefaultValue": 0.0
              }
            }
          ]
        },
        {
          "name": "Attack",
          "type": "State",
          "considerations": [
            { "type": "StateTreeConstantConsideration", "properties": { "Constant": 0.8 } }
          ]
        }
      ],
      "transitions": [
        { "trigger": "OnStateCompleted", "type": "GotoState", "target": "SelectAction" }
      ]
    }
  ]
}
```

### Guard AI with Conditional Transitions
```json
{
  "name": "GuardAI",
  "parameters": [
    { "name": "AlertRadius", "type": "float", "defaultValue": 1000.0 },
    { "name": "VisionAngle", "type": "float", "defaultValue": 90.0 }
  ],
  "states": [
    {
      "name": "Patrol",
      "type": "State",
      "tasks": [...],
      "transitions": [
        {
          "trigger": "OnEvent",
          "type": "EvaluateConditions",
          "conditions": [
            {
              "type": "StateTreeCompareDistanceCondition",
              "properties": {
                "Operator": "Less",
                "Source": "${Context.Target.Location}",
                "Target": "${Context.Owner.Location}",
                "Distance": "${Param.AlertRadius}"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## Editor Workflow

1. Open the LLMStateTree Editor Panel: Window → LLMStateTree Panel
2. Click "Load File" and navigate to `.llmstate` file
3. Click "Generate StateTree" to create the StateTree Blueprint asset
4. Assign the generated StateTree to your AI Controller or Character

## File Location

Place `.llmstate` files in your project's `Config/Examples/` folder for easy access:

```
YourProject/
└── Config/
    └── Examples/
        ├── SimpleAI.llmstate
        ├── GuardAI.llmstate
        └── MyAI.llmstate
```
