# Garage Door - Student Exercise 02

- **Simulation:** Door ex02
- **Platform:** RSLogix 500 
- **Author:** Sam M.
- **Source:** TLP Door Exercises


## Problem Statement (copied verbatim)

> Student Programming Exercise #2:
> 
> In this exercise we want you to apply your knowledge of Relay Logic Instructions to design a program which will maintain the appropriate door movement once initiated by the operator. The Opening or Closing operation of the door will continue to completion even if the operator releases the pushbutton which initiated the movement. The program will adhere to the following criteria:
> 
> 
> 
> Door movement will halt immediately when the Stop Switch is initially pressed, and will remain halted if the switch is released.
> 
> 
> 
> Pressing the Open Switch will cause the door to Open if not already fully open. The opening operation will continue to completion even if the switch is released.
> 
> 
> 
> Pressing the Close Switch will cause the door to Close if not already fully shut. The closing operation will continue to completion even if the Switch is released.
> 
> 
> 
> If the Door is already fully opened, Pressing the Open Switch will Not energize the motor.
> 
> 
> 
> If the Door is already fully closed, Pressing the Close Switch will Not energize the motor.
> 
> 
> 
> Under no circumstance will both motor windings be energized at the same time.
> 
> 
> 
> The Ajar Lamp will be illuminated if the door is NOT in either the fully closed or fully opened position.
> 
> 
> 
> The Open Lamp will be illuminated if the door is in the Fully Open position.
> 
> 
> 
> The Shut Lamp will be illuminated if the door is in the Fully Closed position.
> 
> 
> 
> It is your responsibility to fully design, document, debug, and test your Program. Avoid the use of OTL or OTU latching instructions, ...



## I/O Map 
| Symbol       | Address | Type | Normal     | Description                       |
| ------------ | ------- | ---- | ---------- | --------------------------------- |
| `Open`       | I:1/0   | DI   | Open       | Open (Up) momentary PB            |
| `Close`      | I:1/1   | DI   | Open       | Close (Down) momentary PB         |
| `Stop`       | I:1/2   | DI   | **Closed** | NC Stop PB (opens when pressed)   |
| `LS1`        | I:1/3   | DI   | Open       | Upper limit (door fully **OPEN**) |
| `LS2`        | I:1/4   | DI   | Open       | Lower limit (door fully **SHUT**) |
| `Motor Up`   | O:2/0   | DO   | –          | Motor up windings                 |
| `Motor Down` | O:2/1   | DO   | –          | Motor down windings               |
| `Ajar Lamp`  | O:2/2   | DO   | –          | Indicates door is between limits  |
| `Open Lamp`  | O:2/3   | DO   | –          | Indicates fully open              |
| `Shut Lamp`  | O:2/4   | DO   | –          | Indicates fully closed            |


## Sequence of Operations

1. Open command (maintained to completion):

- Preconditions: not fully open (LS1 is not actuated) and Stop is not pressed.

    - Pressing Open energizes Motor Up. A seal-in using the motor output maintains the run after the PB is released.

    - Motion stops automatically when LS1 actuates (fully open) or if Stop is pressed.

2. Close command (maintained to completion):

- Preconditions: not fully closed (LS2 is not actuated) and Stop is not pressed.

    - Pressing Close energizes Motor Down. A seal-in using the motor output maintains the run after the PB is released.

    - Motion stops automatically when LS2 actuates (fully closed) or if Stop is pressed.

3. Stop behavior:

- Pressing Stop immediately de-energizes whatever motor is running which causes the seal-in to drop out -->  the door remains halted when Stop is released. A new Open/Close command is required to resume.

4. Mutual exclusion:

- Each direction rung has an interlock contact for the opposite motor output so both windings can never be energized at the same time.

    - Position indication:

        - Open Lamp = ON when LS1 is actuated.

        - Shut Lamp = ON when LS2 is actuated.

        - Ajar Lamp = ON when neither LS1 nor LS2 is actuated.

## Safety & Interlocks

- NC Stop (I:1/2) in series at the head of both motor rungs for immediate stop.

- LS1 (XIO) in the Up rung and LS2 (XIO) in the Down rung prevent motors from running at end destination.

- Interlock: Motor Down (XIO) blocks Up, and Motor Up (XIO) blocks Down.

- No OTL/OTU used per the spec.

## State Model 

| State            | Entry Condition (sensed)                                  | Transition (event)                                         | Outputs while in State                            |
| ---------------- | --------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------- |
| **CLOSED**       | `LS2` actuated, `LS1` not actuated                        | **Open** PB → **OPENING**                                  | Motors OFF; **Shut Lamp ON**; Open/Ajar lamps OFF |
| **OPENING**      | Between limits; `Open` command latched by seal-in         | **LS1 actuates** → **OPEN**; **Stop** → **AJAR/STOPPED**   | `Motor Up ON`; interlock prevents Down            |
| **OPEN**         | `LS1` actuated, `LS2` not actuated                        | **Close** PB → **CLOSING**                                 | Motors OFF; **Open Lamp ON**; Shut/Ajar lamps OFF |
| **CLOSING**      | Between limits; `Close` command latched by seal-in        | **LS2 actuates** → **CLOSED**; **Stop** → **AJAR/STOPPED** | `Motor Down ON`; interlock prevents Up            |
| **AJAR/STOPPED** | Between limits; both PBs released (after Stop or release) | **Open** PB → **OPENING**; **Close** PB → **CLOSING**      | Motors OFF; **Ajar Lamp ON**                      |

## Validation

- Maintained Open: From CLOSED, tap Open briefly → door continues to OPEN without holding the PB; verify Motor Up drops at LS1.

- Maintained Close: From OPEN, tap Close briefly → door continues to CLOSED; verify Motor Down drops at LS2.

- Stop Halt & Hold: While OPENING or CLOSING, press Stop → motor de-energizes instantly; release Stop → motion does not resume; require a new command.

- Interlock: Attempt to command the opposite direction while one motor is on → verify the opposite output never energizes.

- PB at Limits: In OPEN, press Open → no motion. In CLOSED, press Close → no motion.

- Lamps: Verify Open Lamp only in OPEN, Shut Lamp only in CLOSED, Ajar Lamp whenever between limits (including Stop-halted mid-travel).