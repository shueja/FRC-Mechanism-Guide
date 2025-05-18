# Rollers

This guide is for rollers or wheels that spin freely (no hard stops) but often push or stall against game pieces or field elements. 

# Conventions

It is important to be consistent regarding which direction is positive. However, best architecture abstracts positive or negative output as task-specific commands (see next section).
> __OPINION__ Generally, positive input should bring the piece into the robot/towards the scoring mechanism. Even if scoring involves feeding the piece out, opposite the direction it came in, scoring should then be negative input.

# Commands
* `Command voltage(DoubleSupplier voltage)`: This is the core command. It runs indefinitely and sets the motor voltage every loop. Other commands can be written using this one.
* `Command stop()`: This is the **default command**. It is `voltage(()->0)`.
* `Command stopOnce()`: This variant sets the motor to 0 volts and ends immediately. It can be used in command compositions (parallel, sequence) without needing extra end conditions.
* Other commands should describe tasks specific to the mechanism's role. For example:
    * Consider a "midtake", which receives and holds a game piece from an intake, but also feeds that game piece to a shooter. It might have:
        * `intake()`, a fast roller speed matching the incoming game piece,
        * `store()`, a slower speed which is used with sensors to put the game piece in a specific position in the midtake,
        * `feed()`, a different speed which pushes the game piece to the shooter
    * Task-specific naming is better than e.g. `fast()`, `slow()` because it allows others writing robot code to understand which action should be used.
* Sometimes the task-specific outputs might be defined outside the roller subsystem. For example, 6995's 2025 code had 6 different ways to score the same game piece (coral). Each scoring option had its own scoring voltage for the "hand" subsystem (not all in the same direction), which was encoded alongside other option-specific configuration. Thus, the scoring command looked something like `hand.voltage(()->scoringOption.scoreVolts)`. This is fine, because the command's role is still self-documenting.

# Control
Use a `VoltageOut` request.

# Constants
It may make sense to store the voltage for each task as a constant, but if the constant is only used in a command factory like
```java
public Command intake() {
    return voltage(()->Constants.INTAKE_VOLTAGE);
}
```
then for easy tunability and understanding of what the command is doing, the voltage value can be written directly in the command factory.
# Configuration
## Current Limits
Current limits are the most important aspect of roller motor configuration. The __stator__ current limit roughly corresponds to maximum applied torque on the roller. 

> Ensure the current limit is enabled: `.withStatorCurrentLimitEnable(true)`. If you are not simulating motor velocity, enable the limit only in real life, with `.withStatorCurrentLimitEnable(RobotBase.isReal())`.

* If the roller needs to stall against a piece continuously, such as to hold a compressible ball, the current limit might need to be high in order to get a solid grip.
* A too-high limit can let the roller break friction and slip past the piece. This may or may not be wanted.
* A too-high limit could also cause mechanical failure if the weak link is a belt run instead of friction against the piece.
* A too-low current limit can show as the roller stopping as soon as it touches a game piece (i.e. any more resistance than just free spinning). 

## Inversion
Configure the InvertedValue so that positive input obeys the convention described above.

# Logging
Logging motor voltage (`motor.getMotorVoltage()`) and stator current (`motor.getStatorCurrent()`) are useful for understanding what the roller was doing in replay, since rollers are otherwise hard to visualize.

# Simulation
Simulation is not necessary for pure voltage control, if the current limit is disabled in simulation.

If the position of the roller becomes needed for visualization, robot behavior, or simulating other sensors, then a `DCMotorSim` can be used.

# Tuning
The primary things to tune are the inversion configuration, the current limit, and the voltages needed for varying tasks, in that order.
