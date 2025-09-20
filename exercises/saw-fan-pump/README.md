# Saw with Fan and Oil Pump   — Master Case Study

**Simulation:** The RSLogix I/O Simulator will be used to represent a Saw with Fan and Oil Pump 
**Platform:** RSLogix 500 
**Author:** Sam M.

## System Overview
Saw Motor, Fan Motor, Pump Motor operator panel with Start/Stop PBs and a Reset Button for the Fan only.

## Problem Statement (Written by Automation instuctor.)
- A saw, a fan, and an oil pump all go on when a start button is pressed. 
- If the saw has operated for more than 20 seconds:
    1. the fan should remain on until a reset by a separate fan reset button
    2. The oil pump should remain on for an additional 10 seconds after the saw is turned off.
- If the saw has operated for less than 20 seconds: 
    1. The oil pump should go off when the saw is turned off.
    2. The fan should run for an additional 5 seconds after the shut down of the saw.



## Assumptions & Clarifications
- Inputs are momentary pushbuttons and outputs are motor coils.
- Stop PB immediately de-energizes saw and clears running timers.
- Fan Reset PB only affects the fan after a long run condition.
- Saw timer uses a TON with 20 second preset
- Pump and fan use TOF timers.


## I/O Map (Master)
| Symbol          | Address | Type | Normal |     Description       |
| ------------    | ------- | ---- | ------ |  ---------------------|
| `Start`         | I:1/0   | DI   | Open   | Saw Start pushbutton  |
| `Stop`          | I:1/1   | DI   | Closed | Saw Stop pushbutton   |
| `Fan Reset`     | I:1/2   | DI   | Open   | Fan Reset pushbutton  |
| `Saw Motor`     | O:2/0   | DO   | –      | Saw Motor output coil |
| `Fan Motor`     | O:2/1   | DO   | –      | Fan Motor output coil |
| `Oil Pump`      | O:2/2   | DO   | –      | Oil Pump output coil  |


## Sequence of Operations (Baseline)
1. If `Saw Motor` runs ≥ 20s:
    - `Stop` pressed → `Fan Motor` latches ON until reset by `Fan Reset`.
    - `Stop` pressed → `Oil Pump` stays on for 10 seconds then off.
2. If `Saw Motor` runs < 20s:
    - `Stop` pressed → `Fan Motor` runs for 5 seconds then stops.
    - `Stop` pressed → `Oil Pump` stops immediatley. 


## Safety & Interlocks
- Seal in circuit for `Saw Motor` requires `Start PB` pressed and S`top PB` not pressed.
- Fan latch uses it's own pushbutton (`Reset Fan`) to for operator control.
- TOF timers prevent conflicts in state e.g. pump will never run longer than logic states.

## State Model
| Saw Runtime  | Saw Motor | Fan Motor       | Oil Pump       |
| ------------ | --------- | --------------- | -------------- |
| Running <20s | ON        | ON              | ON             |
| Stop <20s    | OFF       | ON (5s coast)   | OFF            |
| Running ≥20s | ON        | ON              | ON             |
| Stop ≥20s    | OFF       | Latched (reset) | ON (10s delay) |


## Diagrams
- see ladder diagram in ./saw-fan-pump/ladder

## Validation
1. Test Short Run (<20s): Saw ON 15s → Stop → Fan runs extra 5s → Pump stops immediately.
2. Test Long Run (≥20s): Saw ON 25s → Stop → Fan latched ON until reset → Pump runs 10s then stops.
3. Fan Reset: After long run, press Fan Reset → Fan coil de-energizes immediately.
4. Stop PB override: Press Stop during run → Saw halts, timers drive fan/pump as expected.

## Exports
- Tob be inserted.
