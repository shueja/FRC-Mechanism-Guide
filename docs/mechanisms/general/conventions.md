# Conventions

This section describes measurement conventions used by WPILib and other tools. It also includes other conventions to adopt to make life easier.

## Coordinate System

Reference the [WPILib article](https://docs.wpilib.org/en/stable/docs/software/basic-programming/coordinate-system.html#coordinate-system) on coordinate systems.

> This guide recommends the [Always Blue Origin](https://docs.wpilib.org/en/stable/docs/software/basic-programming/coordinate-system.html#always-blue-origin) field coordinate system, and Phoenix swerve's `FieldCentric` request has the ability to invert controls as described in the WPILib docs.

## Units
WPILib uses meters and radians as the default units for geometry, as it does with most of its math and simulation tooling. For best results, work with distances in a base unit of meters and field angles (such as the heading of the robot or other field positions) in radians.

CTRE Phoenix 6 supports the WPILib Java Units API for configuration. The feedforward/feedback gains and the position/velocity etc reporting is designed to work in mechanism rotations, so the `SensorToMechanismRatio` must be configured correctly. Simulation states work in raw motor rotations, so some unit conversions are required when plumbing simulation. 

For linear mechanisms like elevators, this guide recommends leaving SensorToMechanismRatio as 1 and converting setpoints and feedback between motor rotations and meters in the subsystem. Unfortunately, Phoenix 6's API cannot map motor motion to distance traveled except by pretending one mechanism rotation equals one distance unit (meter).

## Motor
Motors should always be configured so that a positive command causes position feedback to increase and velocity feedback to be positive. Conventions for which mechanism direction is positive are in the individual mechanism guides. Brushless motors do this automatically for built-in encoders, but external encoders, especially those not on the motor shaft, need to be checked for proper inversion relative to the motor.