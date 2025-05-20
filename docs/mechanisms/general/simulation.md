# Simulation

Reference the [Phoenix 6 simulation docs](https://v6.docs.ctr-electronics.com/en/stable/docs/api-reference/simulation/simulation-intro.html)

## Goals of the Simulation Guide
For most teams, simulation only needs to be accurate enough to test subsystem integration logic. This guide will focus on setting up physics models to give subsystems realistic responses. The guide is relatively Phoenix 6 specific because Phoenix 6 simulates the entire motor controller.

## Non-Goals of the Simulation Guide
This guide does not cover replay, in which logged values are fed back into the robot program. For log replay, the most popular option is AdvantageKit. Its [docs](https://docs.advantagekit.org/getting-started/what-is-advantagekit/) explain where replay is useful and how the ability to replay limits code architecture. Without replay, a TalonFX-driven subsystem can have simulation added without having to abstract the hardware access into a real and simulation version.

## Data Flow of Simulation

Consider a position-based subsystem which controls and gets feedback from a motor:

```java
public class PositionSubsystem extends SubsystemBase {
    TalonFX motor = ...;
    
    /** Requests on-motor-controller feedback*/
    private void goToPosition(double position) {...}
    public double getPosition() {...}
}
```
Consider the real-life cycle of information in this subsystem.
1. `goToPosition` sends a setpoint to the motor controller,
2. which executes a control loop based on the current position.
3. As the motor applies torque, the physical robot moves in a predictable way.
4. The motor measures the position as it changes using the encoder.
5. The position is used in the motor's control loop and also sent back to the robot code.

How can this process be run without real hardware? If the program running on the motor controller is simulated accurately, as Phoenix 6 is designed to do, then only step 3 needs to be different between simulation and real-life.

Note that step 3 assumes the robot moves in a predictable way. Based on measured physical properties of the mechanism or the motor's own tuned feedforward constants (`kS`, `kV`, `kA`, and `kG` where applicable), we can write calculations which model the next mechanism state (such as position and velocity) given the current state and the voltage input. This is the core of physics simulation: reading what the motor would be doing, calculating its effect, and telling the (simulated) motor controller what actually happened.

> __TERMINOLOGY__
>
> * The **state** in FRC usage usually consists of both the position and velocity of the mechanism. Some WPILib simulation classes use a state that is just velocity.
> * The **input** is the voltage exerted by the motor. This is an "output" of the robot code, but an input to the physics-modeling equations.
> * The **model** is the set of equations that describe the next state in terms of the current state and input.

## Implementation

Where should the simulation code go?

Logically, the code that simulates reality should be separate from the rest of your robot code. It should only interface with the subsystem code through information read from and provided to the motor controller. Some uses of simulation in industry have the simulation code in an entirely separate program, created with tools designed to model physical systems.

However, in FRC usage, the simulation still needs to access the motor class, be set up using the subsystem's constants, and run in the periodic loop during simulation. For ease of development, simulation code can just go in the subsystem's `simulationPeriodic` method.

### Physical Properties or Feedforward Constants?

The subsystem should have a sim object within it, which is constructed using physical properties or feedforward constants and keeps track of the simulated system state. While constructing with physical properties such as mass, inertia, and gear ratio may seem appealing, especially before the hardware is available, keep in mind that after tuning the mechanism, feedforward constants will give a much more accurate model of the system than estimated physical properties. For early development, the mechanism calculator [Recalc](reca.lc) can convert estimated physical properties to theoretical `kV` and `kA`.

### The Sim Model Class

 These sim objects can also model hard range limits and account for gear ratios. WPILib provides several sim objects for common use cases. They differ in the units used for configuration and output:

|Class|State|Units|Notes|Usage
|-----|-----|-----|---|---|
|DCMotorSim|Position, Velocity|Motor Rotations|No range limits|Rollers, Flywheels
|ElevatorSim|Position, Velocity|Meters|Range limits, gravity modeling (assumes vertical elevator)|Elevators, range-limited linear motion
|SingleJointedArmSim|Position, Velocity|Radians|Range limits, gravity modeling (assumes 0 is arm horizontal)|Arms, wrists, pivoting intakes, range-limited angular motion


### DCMotorSim Code Example

This example shows how simulation can be plumbed and how the same constants can be used in motor controller feedforward and in simulation setup. This example can be used for rollers or flywheels.

`ElevatorSim` and `SingleJointedArmSim` have different constructors and interfaces. See the simulation examples in [Elevators](../elevators/#simulation) and [Pivots](../pivots/#simulation).

The motor is assumed to have its SensorToMechanismRatio (and RotorToSensorRatio for external feedback) set so that gains and feedback are in mechanism rotations. 
```java
// TalonFX configs, probably not stored in the subsystem directly
// motor rotations per mechanism rotation, >1 for reductions
private final double GEARING =
    config.Feedback.SensorToMechanismRatio * config.Feedback.RotorToSensorRatio;
private final Slot0Configs gains = ...

// Gains need to be converted from mechanism rotations to motor radians
// V*s/(mechanism rotations) / (gearing * 2pi) = V*s^2/(motor radians)
private final DCMotorSim motorSim =
    new DCMotorSim(
        LinearSystemId.createDCMotorSystem(
            gains.kV / (GEARING * 2 * Math.PI),
            gains.kA / (GEARING * 2 * Math.PI)),
        DCMotor.getKrakenX60(1));

public void simulationPeriodic() {
// Get the SimState object
var simState = motor.getSimState();
// Modeling voltage dips is generally overkill for testing integration
simState.setSupplyVoltage(12);
// simState.getMotorVoltage is counterclockwise negative
double volts = simState.getMotorVoltage();
// Subtract out the kS term, since DCMotorSim doesn't include it in the model.
if (Math.abs(voltage) <= gains.kS) {
    volts = 0;
} else {
    volts -= Math.copySign(gains.kS, volts);
}
motorSim.setInput(volts);
// Calculate one loop time into the future.
motorSim.update(0.02);
var rotorPos = motorSim.getAngularPositionRotations();
var rotorVel = motorSim.getAngularVelocityRPM() / 60.0;

simState.setRawRotorPosition(rotorPos);
simState.setRotorVelocity(rotorVel);
}


