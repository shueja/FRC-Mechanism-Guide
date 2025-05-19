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

    /** Returns if a piece is currently detected*/
    public boolean pieceDetected() {...}

    /**
     * Suppose the sensor needs some debouncing; this can be done by
     * wrapping pieceDetected in a Trigger.
     */
    public final Trigger pieceDetectedTrg =
        new Trigger(this::pieceDetected).debounce(0.1);

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
            receive().until(pieceDetectedTrg)
            // if piece is detected when storePiece starts, skip the receive
                .unless(pieceDetectedTrg),
            // reverse slowly until no longer detecting
            voltage(-0.5).until(pieceDetectedTrg.negate()),
            //The piece is now behind the beam, not breaking it.
            voltage(0.2).until(pieceDetectedTrg),
            stopOnce()
        ).withName("storePiece");
    }
}

// Usage elsewhere
void bindButtons() {
    Trigger intakeButton = controller.a();
    // Command factories can be called during code setup.
    intakeButton.onTrue(intake.intake()).onTrue(conveyor.storePiece());
}

```
