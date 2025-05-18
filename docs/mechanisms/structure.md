# How to Read This Guide

Each mechanism's page has several sections:

## Conventions

Consistency is key to ease of integration, collaboration, and modification. This section will describe the relevant conventions used by the software tools available (WPILib, Phoenix 6). These conventions define what direction is positive and what the zero point is for measuring the mechanism. For best experience, your code should align with those tools.

## Commands
Suggests the externally accessible options for interacting with the mechanism. These take the form of `public` methods in the subsystem which return simple `Command`s that require the subsystem. By exposing these commands, the subsystem developer gives other developers a clear definition of and safe access to the subsystem's capabilities.

## Control
Details how the commands can be implemented with Phoenix 6 `ControlRequest`s.

## Configuration
Details elements of motor configuration which are important for each use case.

## Constants
Suggests ways to organize constants such as the physical properties of the mechanism or important output values that need names.

## Logging
Suggests feedback from the motor or other sources that would be useful to log or broadcast for telemetry. 

## Feedback
Suggests feedback from the motor or other sources that is useful for automation.

## Simulation
Suggests what parts of the physics of the mechanism are worth modeling and how, and what parts aren't likely to be useful given the use case of simulation.

## Tuning

Suggests a process for tuning the software, starting from the first time the hardware is available. This includes checks to do before even enabling, in order to minimize risk of easily-catchable errors leading to mechanism damage.