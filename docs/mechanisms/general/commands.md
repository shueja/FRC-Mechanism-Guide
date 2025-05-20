# Commands
These guides use the __Subsystem Command Factory__ architecture within the WPILib command-based framework. This is recommended by the designers of the command framework for several reasons:

## Architecture Goals

1. Command-based programming is designed to prevent hardware (such as motors) from receiving and trying to follow two conflicting instructions at the same time. 
2. To best enforce this practice, a subsystem should make it impossible for code outside the subsystem to control the hardware outside of a command that requires the subsystem.
3. Commands representing the most basic subsystem actions usually only need one or two of the command lifecycle stages (initialize, execute, isFinished, end)
4. It is helpful to be able to tell what basic commands a subsystem can perform from within an autocomplete list.
5. As required by the command scheduler, a single instance of a command should not be used in more than one composition. The architecture should make it easy to get a new instance of a given command each time the command is needed.

## Subsystem Command Factory

* The subsystem's only public methods for performing actions are Command Factories.
    * These methods are usually called during code setup, for example when binding buttons or constructing autonomous routines.
    * The methods take some parameters and return Commands requiring the subsystem (and no other subsystems)
    * Every time the factory method is run, it constructs and returns a new Command instance.
* The subsystem can and should still have non-controlling methods that expose feedback about the subsystem's state, such as `getPosition()` or `boolean isAtTarget(double target)`.
* Private methods that directly control hardware are a normal implementation detail, but they must be private so they cannot be used from outside the subsystem.

## Implementation Techniques
The code example below uses several techniques designed to help implement this architecture.

* Use of `Subsystem.run()/runOnce()/startEnd()/startRun()` to easily wrap a private subsystem method into a command which automatically requires the subsystem.
    * For clarity, the example here calls these as `this.run(...)` etc, but simply calling `run(...)` in a subsystem has the same effect.
* Use of command decorators such as `until(BooleanSupplier condition)` and the inline commands in `edu.wpi.first.wpilibj2.command.Commands`.
* Exposing boolean-returning methods as `Trigger`s for convenient combination with other conditions.
* Method reference syntax `object::method` as a shorthand for `()->{return object.method();}` 


## Subsystem Example
This example is of a conveyor with a game piece detection sensor, such as a beam break. This would be classed as a "roller" subsystem. See the [Rollers guide](/mechanisms/rollers) for details on implementing subsystems like this.

```java
// for sequence(), etc
import static edu.wpi.first.wpilibj2.command.Commands.*; 
public class Conveyor extends SubsystemBase {
    public Conveyor() {...}

    /**
     * Suppose the sensor needs some debouncing; this can be done easily when
     * using a Trigger
     */
    public final Trigger pieceDetected =
        new Trigger(()->{/*beam broken*/}).debounce(0.1);

    /**
     * This method would be called each loop, so the parameter is not a Supplier.
     * It is private, according to the architecture.
     */
    private void setVoltage(double volts) {...}

    public Command voltage(double volts) {...}

    /**
     * This matches the speed the piece would have entering the conveyor.
     */
    public Command receive() {
        return voltage(10).withName("receive");
    }

    public Command stopOnce() {
        return this.runOnce(()->setVoltage(0));
    }

    /**
     * Even complex commands can be defined inside the subsystem.
     * This one tries to move the conveyor so the piece is barely forward enough to break the beam.
     * It assumes the piece is either currently breaking the beam, behind the beam, or not yet in the conveyor.
     */
    public Command storePiece() {
        return sequence( // imported from Commands.*
            // Run quickly forward until the piece is detected.
            receive().until(pieceDetected)
            // if piece is detected when storePiece starts, skip the receive
                .unless(pieceDetected),
            // reverse slowly until no longer detecting
            voltage(-0.5).until(pieceDetected.negate()),
            //The piece is now behind the beam, not breaking it.
            //Move forward slowly to position it precisely, barely breaking the beam.
            voltage(0.2).until(pieceDetected),
            stopOnce()
        ).withName("storePiece");
    }
}

// Outside the subsystem
void bindTriggers() {
    Trigger intakeButton = controller.a();
    // Command factories should be called during code setup, not when trying to run the command.
    intakeButton.onTrue(intake.intake()).onTrue(conveyor.storePiece());
    // Using the Trigger outside the subsystem.
    conveyor.pieceDetected.whileTrue(/*driver feedback*/);
}

```

## Alternative Command Architectures

### Problems with Subsystem Command Factories

The above architecture does have some downsides:

* Sometimes a subsystem has so many command factories or factories with enough complexity that it becomes hard to navigate or find the internal implementation of the subsystem. 
* When creating a command that only requires one subsystem but needs information from another (such as if `pieceDetected` was not within the `Conveyor` class), a subsystem command factory needs to have that information passed in as a parameter. This is known as dependency injection (DI). Alternatively, the command factory can be written somewhere where both subsystems can be injected or are already in scope.  
* Factories creating multi-subsystem compositions have the same issues regarding DI, and really shouldn't be in any one subsystem. Writing full compositions as-needed (i.e. within a button binding) works but makes it hard to reuse work, such as between teleop bindings and auto routines.

Below are a few other common ways to organize the definitions of commands.

### Separate Factory Classes

Writing non-subsystem classes containing multi-subsystem factories is a great way to organize the robot's higher-level subsystem integration. There are generally two ways to do this.

1. Take an instance of each subsystem in the class constructor and access the class's member variables in the subsystems. This is less boilerplate and leads to compositions looking similar between the factory class and `Robot.java`.
2. Take an instance of each needed subsystem in each factory's parameter list, so that the factories can be static. This is more typing when calling each factory, but makes it easier to spread factory methods out wherever they make sense to write. 

### Class Commands

WPILib does support writing full subclasses of `Command` to define command behavior. This is __not generally recommended__ because most subsystem behavior can be expressed more succinctly with compositions of factory methods. However, class commands have some benefits:

* They can easily store state that might need to be captured on `initialize`, during some `execute`s, etc.
* When the command has many parameters or different possible constructors, it can help to write them as class constructors rather than factory methods within the subsystem. 
* If all four lifecycle methods have complicated bodies that aren't used in other commands or exposed in the subsystem, a class command can contain those long methods.

When using class commands, ensure the following:

1. You MUST call `addRequirements(Subsystem...)` in the constructor with any subsystems required by the command. The automatic requirement implicit in the `Subsystem#run()` etc. factories is __not__ available here.
2. You are not creating and scheduling other, more simple subsystem commands. This will cause the class command to be interrupted.

### Package-Private: Preserving Access Safety Outside Subsystems

When defining commands outside the subsystem which directly call the hardware-controlling methods, the prior advice of making the hardware-controlling methods private does not work. However, Java has the concept of __package-private__ methods, which uses the package layout (defined by the directory layout) to restrict access.

Consider the following directory layout:
```
robot
| subsystems
| | conveyor
| | | Conveyor.java (frc.robot.subsystems.conveyor.Conveyor)
| | |_ReceiveCommand.java (frc.robot.subsystems.conveyor.ReceiveCommand)
| |_MultiSubsystemFactories.java (frc.robot.subsystems.MultiSubsystemFactories)
|_Robot.java (frc.robot.Robot)
```

`Conveyor` has a package-private `setVoltage` method, declared by the lack of another access modifier:

```java
public class Conveyor extends SubsystemBase {
    void setVoltage(...){...}
}
```

If `ReceiveCommand` can access an instance of `Conveyor`, it can call the package-private `setVoltage`:
```java
public class ReceiveCommand extends CommandBase {
    private Conveyor conveyor;
    public ReceiveCommand(Conveyor conveyor) {
        this.conveyor = conveyor;
    }
    @Override
    public void initialize() {
        conveyor.setVoltage(10);
    }
}
```

But `Robot` cannot call `setVoltage` because `frc.robot.Robot` is not in the same package as `frc.robot.subsystems.conveyor.Conveyor`:

```java
public class Robot extends TimedRobot {
    private Conveyor conveyor = new Conveyor();
    void teleopPeriodic() {
        conveyor.setVoltage(10); // This ERRORS, as it should.
    }
}
```
