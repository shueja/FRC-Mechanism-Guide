# FRC Software Mechanism Design Patterns

FRC mechanisms fall into a few common categories. The needs and implementations of each category of mechanisms often have many common key details, but differ too much to directly copy-paste or abstract into a multi-season library. This site intends to capture the key details in order to help guide controls implementation for a given mechanism.

## Who This Is For

The author is a former mentor and former student on team 6995 NOMAD. The advice in this site is based on the author's experience up to and through the 2025 season and is written with team 6995's situation in mind. The guides assume:

* Java command-based architecture
* Use of command factory methods in subsystems and for compositions (TODO a page explaining these patterns)
* Phoenix 6 (TalonFX) is used for all mechanisms

The content on this site is the opinion of the author and is not official training material of 6995 NOMAD, nor does it necessarily represent 6995's continued practices.

## What This Is Not

This is not a substitute for basic Java tutorials, WPILib documentation, or motor vendor documentation. This content assumes a basic familiarity with Java, the command-based architecture, and the various control strategies, configuration options, and feedback provided in the Phoenix 6 TalonFX library.

## Guides for Mechanism Types

<div class="grid cards" markdown>

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

-   [__LED Strips__](./mechanisms/leds.md)

    ---

    Light strips that signal the state of the robot.

</div>
