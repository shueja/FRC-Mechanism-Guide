# FRC Software Mechanism Design Patterns

These guides are for FRC programmers tasked with creating a subsystem for a particular kind of mechanism. FRC mechanisms fall into a few common categories, and this site captures techniques and general advice for each category. 

These guides are highly opinionated and based on team 6995 NOMAD's 2025 season codebase and prior experience. However, parts of the guides are still helpful under other software architectures.

The guides assume:

* Java command-based architecture
* Use of command factory methods in subsystems and for compositions (TODO a page explaining these patterns)
* Every mechanism is driven by a TalonFX with Phoenix 6
    * TalonFXS is largely the same, but the author does not have experience with TalonFXS and its need for an external encoder. 
* Pneumatics are not included 

## How to Use this Guide

1. Each guide page has several categories of advice; see [here](./mechanisms/structure) for an explanation of each category.
2. Start by reading the [general advice](./mechanisms/general) that applies to any kind of motorized subsystem.
3. Read the page specific to your category of mechanism below.

## Guides for Mechanism Types

[__How Each Guide is Structured__](./mechanisms/structure.md)

<div class="grid cards" markdown>

-   [__General__](./mechanisms/general.md)

    ---

    Advice applicable to all motorized subsystems.

-   [__Rollers__](./mechanisms/rollers.md)

    ---

    Any mechanism that spins at a non-precise speed, with no hard limits to its range of motion.


-   [__Manual or End-to-End Position Mechanisms__](./mechanisms/manual.md)

    ---
    Position control by manual (joystick/button) input or driving to a hard stop.
    


-   [__Elevators__](./mechanisms/elevators.md)

    ---

    Feedback-controlled linear position, with or without gravity compensation.



-   [__Pivots__](./mechanisms/pivots.md)

    ---

    Arms, Wrists, Turrets, Swerve Steering, with or without gravity compensation.


-   [__Flywheels__](./mechanisms/flywheels.md)

    ---

    Usually in shooting games, using feedback to get consistent wheel speed. Also Swerve Drive motors.



-   [__Swerve__](./mechanisms/swerve.md)

    ---

    Suggestions for building on the CTRE swerve library.

</div>

## What This Is Not

This is not a substitute for basic Java tutorials, WPILib documentation, or motor vendor documentation. This content assumes a basic familiarity with Java, the command-based architecture, and the various control strategies, configuration options, and feedback provided in the Phoenix 6 TalonFX library.

The content on this site is the opinion of the author and is not official training material of 6995 NOMAD, nor does it necessarily represent 6995's continued practices.