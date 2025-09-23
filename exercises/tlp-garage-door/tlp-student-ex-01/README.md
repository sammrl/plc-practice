# Garage Door Ex01  

**Simulation:** Door ex01
**Platform:** RSLogix 500 
**Author:** Sam M.


## Problem Statement (copied verbatim)
> In this exercise we want you to apply your knowledge of Relay Logic Instructions to design a program which will control the LogixPro Door Simulation. The Door System includes a Reversible Motor, a pair of Limit Switches and a Operator Control Panel, all connected to your PLC. The program you create will monitor and control this equipment while adhering to the following criteria:
> 
> 
> 
> In this exercise the Open and Close pushbuttons will be used to control the movement of the door. Movement will not be maintained when either switch is released, and therefore the Stop switch is neither required nor used in this exercise. However, all other available Inputs and Outputs are employed in this exercise.
> 
> 
> 
> Pressing the Open Switch will cause the door to move upwards (open) if not already fully open. The opening operation will continue as long as the switch is held down. If the switch is released, or if limit switch LS1 opens, the door movement will halt immediately.
> 
> 
> 
> Pressing the Close Switch will cause the door to move down (close) if not already fully closed. The closing operation will continue as long as the switch is held down. If the switch is released, or if limit switch LS2 closes, the door movement will halt immediately.
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
> The Open Lamp will be illuminated if the door is in the Fully Open position.
> 
> 
> 
> The Shut Lamp will be illuminated if the door is in the Fully Closed position.

## I/O Map (Master)
| Symbol          | Address | Type | Normal |       Description                        |
| --------------- | ------- | ---- | ------ |  --------------------------------------- |
| `Open`          | I:1/0   | DI   | Open   | Motor 1 (up windings) Start pushbutton   | 
| `Close`         | I:1/1   | DI   | Open   | Motor 2 (down windings) Start pushbutton |
| `LS1`           | I:1/3   | DI   | Open   | Upper Limit Switch                       |
| `LS2`           | I:1/4   | DI   | Open   | Lower Limit Switch                       |
| `Motor Up`      | O:2/0   | DO   | –      | Motor 1 (up windings) output coil        |
| `Motor Down`    | O:2/1   | DO   | –      | Motor 2 (down windings) output coil      |
| `Open Lamp`     | O:2/3   | DO   | –      | Open Lamp output coil                    |
| `Shut Lamp`     | O:2/4   | DO   | –      | Shut Lamp output coil                    |


## Sequence of Operations
1. If door is not fully open:
    - `Open` pressed → `Motor Up` energized as long as `Open` is pressed.
    - The door continues opening until `Open` is released OR `LS1` loses physical contact with the door (this is the door's fully open position.)
2. If door is not fully closed:
    - `Close` pressed → `Motor Down` energized as long as `Close` is pressed.
    - The door continues closing until `Close` is released OR `LS2` makes physical contact with the door (this is the door's fully closed position.)


## Safety & Interlocks
- Safety interlock - XIO `Motor Down` and XIO `Motor Up` - prevents motors from running at same time.
- Limit Switches used to sense door position - `LS1` and `LS2` act as guards from door proceding past the fully open or fully closed position, respectivley. 

## State Model
| State            | Entry Condition (sensed)                                | Allowed Command & Transition                                                                          | Outputs while in State                                                           |
| ---------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **CLOSED**       | At lower limit (**LS2 actuated**); not at upper limit   | **Open** PB held → **OPENING**. Close PB has **no effect**.                                           | `Motor Up`=OFF, `Motor Down`=OFF, **Shut Lamp=ON**, Open Lamp=OFF                |
| **OPENING**      | Between limits; **Open PB held**; interlock blocks Down | **Release Open** PB → **AJAR/STOPPED**. **Reach upper limit (LS1 actuated as specified)** → **OPEN**. | `Motor Up`=ON, `Motor Down`=OFF, Lamps per limit (both OFF while between limits) |
| **OPEN**         | At upper limit (**LS1 actuated per spec**)              | **Close** PB held → **CLOSING**. Open PB has **no effect**.                                           | `Motor Up`=OFF, `Motor Down`=OFF, **Open Lamp=ON**, Shut Lamp=OFF                |
| **CLOSING**      | Between limits; **Close PB held**; interlock blocks Up  | **Release Close** PB → **AJAR/STOPPED**. **Reach lower limit (LS2 actuated)** → **CLOSED**.           | `Motor Down`=ON, `Motor Up`=OFF, Lamps per limit (both OFF while between limits) |
| **AJAR/STOPPED** | Between limits; both PBs released                       | **Open** PB held → **OPENING**. **Close** PB held → **CLOSING**.                                      | `Motor Up`=OFF, `Motor Down`=OFF, Lamps OFF (no limit reached)                   |



## Validation


- Open from Closed: Start in CLOSED (Shut Lamp ON). Hold Open → Motor Up energizes; releasing Open stops the door immediately. Re-hold Open to continue; when upper limit actuates (per spec, LS1), motor de-energizes and state = OPEN (Open Lamp ON).

- Close from Open: From OPEN, hold Close → Motor Down energizes; releasing Close stops movement. Re-hold Close to continue; when lower limit (LS2) actuates, motor de-energizes and state = CLOSED (Shut Lamp ON).

- Interlock Check: While OPENING, press Close simultaneously → verify Motor Down never energizes (and vice-versa while CLOSING). At no time are both windings ON.

- Between-Limits Control: Place door mid-travel.

    - Verify:

        - Hold Open → enters OPENING and runs only while held.

        - Release → stops.

        - Hold Close → enters CLOSING and runs only while held.

        - Lamps: Verify Open Lamp ON only in OPEN; Shut Lamp ON only in CLOSED; both OFF between limits.
