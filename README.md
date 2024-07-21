# Search-and-Rescue Simulation

## Project Overview

This repository contains a UPPAAL model for simulating search-and-rescue scenarios. The goal is to represent a dangerous environment with civilians needing rescue, first responders attempting to save them, and drones dispatching instructions to the civilians.

## Contents
1. [Design](#design)
2. [Model Description](#model-description)
3. [System Configuration](#system-configuration)
4. [Testing Scenarios](#testing-scenarios)
5. [Properties](#properties)
6. [Verification Results](#verification-results)
7. [Conclusions](#conclusions)

## Design
### Area Layout
The grid is a rectangular layout where:
- No two drones can occupy the same cell.
- Drones are assigned specific columns based on their IDs.
- Exits are placed at the edges of the grid.
- Civilians move towards the nearest edge to find exits.

### Stochastic Features
The model includes:
- Drone sensor failure rates (`Pf`).
- Human instruction acknowledgment rates (`Pr`).

### Design Assumptions
1. No two entities (civilian or first responder) can occupy the same cell.
2. Entities cannot occupy cells with fire or exits.
3. Distances are measured using the Manhattan distance.
4. Survivors follow drone instructions.

## Model Description
### High-Level Description
The model is composed of four entities:
- **Civilian**: Must be saved, can save themselves by reaching an exit, or help others when instructed by a drone.
- **Drone**: Patrols the grid and communicates with civilians to assist others.
- **First Responder**: Navigates the grid to assist individuals in need.

### Component Description
#### Civilian
- **InNeed**: Requires assistance if near fire. Will die if not helped within `Tv` time units.
- **Survivor**: Moves towards the nearest edge but will help others if instructed by a drone.
- **ZeroResponder**: Stays in position until the person in need reaches them, then takes `Tzr` time units to save them. If unsuccessful, reverts to survivor status.
- **CallingFirstResponder**: Calls a first responder to help a person in need, waits for the responder to arrive.

#### Drone
- Moves until it detects a person in need and a survivor within `Nv` distance units. Instructs survivors to help or call first responders as needed.

#### First Responder
- Assists nearby civilians, responds to survivor calls, or moves towards the center of the grid.

## System Configuration
### System Parameters
| Parameter      | Description                                    |
|----------------|------------------------------------------------|
| `GRID_W`       | Grid width                                     |
| `GRID_H`       | Grid height                                    |
| `N_CIVILIANS`  | Total number of civilians                      |
| `N_DRONES`     | Total number of drones                         |
| `N_FR`         | Total number of first responders               |
| `Tv`           | Time for a civilian to die near fire           |
| `global_time`  | Absolute time of the system                    |
| `grid`         | Matrix representing the scenario               |
| `pos`          | Array of coordinates of civilians and responders|
| `posDrone`     | Array of coordinates of drones                 |
| `helping`      | Coordinates of persons assisted by zero responders |
| `calling`      | Coordinates of first responders contacted by zero responders |
| `FRHelping`    | Coordinates of persons assisted by first responders |
| `n_safe`       | Number of civilians saved                      |
| `Tscs`         | Time to evaluate properties                    |
| `N_perc`       | Percentage of civilians safe to evaluate       |

### Channels
- `starting`: Used by all entities to start simultaneously.
- `assistDone`: Synchronizes survivors and responders/people in need.
- `dead`: Signals a person in need has died.
- `callFirstResponder`: Drones use to instruct survivors to contact responders.
- `assist`: Drones use to instruct survivors to assist people in need.

### System Setup
1. Configure the grid in `initializeGrid()`.
2. Place entities using `placeEntity()`.
3. Initialize variables with `initCallingHelping()`, `initFRHelping()`, and `n_safe=0`.
4. Set system parameters in the 'System Declarations' file.

## Testing Scenarios
### Grid Layouts
Three layouts simulate different real-world scenarios:
1. **Scenario 1**: Corridor on fire with an emergency exit.
2. **Scenario 2**: Enough first responders to help civilians.
3. **Scenario 3**: Larger grid requiring survivor collaboration.

### Testing Parameters
| Scenario  | Grid Size | Civilians | Drones | First Responders | `Tv` |
|-----------|-----------|-----------|--------|------------------|------|
| Scenario 1| 3x7       | 4         | 1      | 2                | 13   |
| Scenario 2| 6x5       | 4         | 1      | 3                | 4    |
| Scenario 3| 8x8       | 5         | 2      | 1                | 10   |

## Properties
### Deterministic Version
1. **Property 1**: Checks if at least `N%` of civilians are safe within `Tscs` time units.
2. **Property 2**: Ensures `N_perc` of civilians are always safe within `Tscs` time units.
3. **Property 3**: Guarantees `N_perc` of civilians are safe in all states when `Tscs` is reached.

### Stochastic Version
1. **Property 1**: Checks the probability of at least `N%` of civilians being safe within `Tscs`.
2. **Property 2**: Checks the probability of `N_perc` of civilians being safe when `Tscs` is reached.
3. **Property 3**: Evaluates the expected value of `n_safe`.

## Verification Results
### Deterministic Version
| Query     | Scenario 1 | Scenario 2 | Scenario 3 |
|-----------|------------|------------|------------|
| Query 1   | Satisfied  | Satisfied  | Satisfied  |
| Query 2   | Satisfied  | Satisfied  | Satisfied  |
| Query 3   | Satisfied  | Satisfied  | Unsatisfied|

### Stochastic Version
Three scenarios test different probabilities and grid configurations, analyzing the probability and consistency of civilians being saved.

## Conclusions
The model provides a valuable tool for evaluating and optimizing search-and-rescue scenarios. By adjusting entity positions and parameters, it can help ensure the overall effectiveness and robustness of safety systems. However, real-world behavior may vary due to unpredictable external events.

---

For detailed analysis and results, refer to the full report found at .
