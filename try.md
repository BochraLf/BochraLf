# Agent Zork 

An AI agent designed to play text-based adventure games (e.g., Zork, Lost Pig) using a hybrid architecture combining:**FastMCP** ,**Qwen2.5-72B** and **Jericho API**. Presenting the progressive Improvements i did: 

## 🔹 Solution 0 — Baseline

### Approach

- Standard ReAct agent
- Tools:
  - `play_action`
  - `memory`
  - `get_map`
  - `inventory`
- Basic loop detection:
  - Same action repeated 3 times → force `"look"`

### Result


Score: 2
Moves: 96
Locations: 25


Baseline reference.



## 🔹 Solution 1 — Improved Single Agent

### Improvements

- Aggressive loop detection: `recent_actions[-3:]`
- Introduced `tried_actions` set to track attempted actions
- Automatic alternative action when repetition detected

### Result


Score: 1
Moves: 99
Locations: 37


✔ Good exploration 
✖ Lower score



## 🔹 Solution 2 — Agent + Rewritten Server

### Agent Changes

- `room_exits_tried` tracking per room
- Prompt includes untried directions

### Server Changes

- Map annotates rooms as `"NOT YET EXPLORED"`

### Result


Score: 1
Moves: 96
Locations: 15


⚠ Exploration regressed



## 🔹 Solution 3 — Game-Specific Hints

Added hints inside the system prompt:

> In Fountain Room: examine the bowl → coin inside

Goal: guide the agent toward point-scoring actions.

### Result


Score: 2
Moves: 99
Locations: 26


✔ Score improved  
✖ Exploration limited


## 🔹 Solution 4 — Stricter Loop Detection

### Changes

- Reduced hint verbosity:
  - `"examine bowl ONCE then LEAVE"`
- Loop detection changed to:
  - `recent_actions[-2:]`

### Result


Score: 2
Moves: 100
Locations: 29


✔ Best balance between score and exploration (so far)



## 🔹 Solution 5 — Map Injection Strategy

### Changes

- Automatic map injection every 8 steps
- Prompt includes untried directions per room
- Server marks `"NOT VISITED YET"` rooms

### Result


Score: 2
Moves: 95
Locations: 14


⚠ Map injection ineffective



## 🔹 Solution 6 — Deterministic Navigation (Reduced LLM)

### Philosophy

Call the LLM only when:

- No unexplored exits remain
- A puzzle is detected

Navigation handled in pure Python:

- Room graph construction
- Exit tracking
- Automatic object interaction

### Result


Score: 1
Moves: 100
Locations: 27


⚠ Reducing LLM did not improve results



## 🔹 Solution 7 — Jericho API Integration

### Improvements

- Use `location_id` (unique physical room ID)
- Use `get_valid_actions()` for full action space
- Log actions per location
- Extract promising actions from observation
- Add anti-stagnation bias

### Result


Score: 1
Moves: 99
Locations: 20


⚠ Jericho alone was insufficient

## Summary of Results

| Solution | Score | Moves | Locations | Notes |
|----------|--------|--------|------------|-------|
| Sol 0 | 2 | 96 | 25 | Baseline |
| Sol 1 | 1 | 99 | 37 | Best exploration |
| Sol 2 | 1 | 96 | 15 | Regression |
| Sol 3 | 2 | 99 | 26 | Hints added |
| Sol 4 | 2 | 100 | 29 | Best balance |
| Sol 5 | 2 | 95 | 14 | Map ineffective |
| Sol 6 | 1 | 100 | 27 | Deterministic |
| Sol 7 | 1 | 99 | 20 | Jericho API |


## 🏆 Solution 8 — Final Hybrid Architecture

### Concept

**Server:** Based on Solution 6  
**Agent:** Fusion of Solution 1 + Solution 4



##  Two-Phase Strategy

### Phase 1 — Interact First

When entering a new room:

1. Interact with objects (`take`, `open`, `examine`)
2. Execute forced scoring actions

### Phase 2 — Explore Exits

Try all exits systematically.



#  Targeted Deterministic Fix

Problem observed:

- Agent explored extremely well (53+ locations)
- Missed scoring actions (e.g., did `examine fountain` but not `examine bowl`)

### Solution

Introduce hardcoded forced actions.



# Final Architecture Details (Sol.08) 

## agent.py

### ROOM_FORCED_ACTIONS

```python
ROOM_FORCED_ACTIONS = {
    "Fountain Room": ["examine bowl", "take coin"],
    "Hole": ["take torch"],
}
```

Ensures scoring actions are executed immediately upon entering specific rooms.

### Forced Action Tracking
```python
self.forced_done: set[str] = set()

```

Prevents repeating forced actions.

### Strict Loop Detection
```python
if action in self.recent_actions[-2:]:

```

Cuts loops earlier than baseline.

### Exit Tracking
```python
self.room_exits_tried: dict[str, set[str]] = {}

```

Tracks which directions were attempted per room, and major reason behind improved exploration.

## mcp_server.py
### get_state()

Returns structured JSON:
```json
{
    "location_name": ...,
    "location_id": ...,
    "visits": ...,
    "exits": ...,
    "objects": ...,
    "valid_actions": ...
}
```

### improvements done:

- `location_id` ensures accurate room tracking

- `valid_actions` from Jericho provides full action space

- Objects extracted for interaction prioritization

### Automatic Exit Graph Construction
```python
if action in DIRS and new_location != self.current_location:
    self.room_exits[self.current_location][action] = new_location
```

Builds the map dynamically.

**Final Results!!** After deterministic forced-action integration:


🎆🎆🎆🎆🎆🎆🎆🎆🎆


- Score: 2
- Moves: 98
- Locations: 61

  
🎆🎆🎆🎆🎆🎆🎆🎆🎆



END. Bochra LAFIFI
