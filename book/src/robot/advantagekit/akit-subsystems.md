# Subsystems With AdvantageKit

Building off of both the previous section, and the section on IO Layers, we can now create a subsystem that uses an IO
layer to interact with hardware. This allows us to easily swap out the implementation of the hardware without changing
the
code that uses it, and makes it easy to test the subsystem in isolation with a mock implementation of the IO layer.

## Creating a Subsystem

The structure of a subsystem using AdvantageKit is very similar to that of a subsystem using barebones WPILib. The main
difference is that instead of directly interacting with the hardware, you interact with an IO layer that abstracts away
the
hardware. This makes it easy to swap out the implementation of the hardware without changing the code that uses it, and
makes
it easy to test the subsystem in isolation with a mock implementation of the IO layer.

A subsystem in AdvantageKit typically won't contain any methods for interacting with the hardware, instead, it will
contain
a public instance of an IO layer for raw hardware interaction from outside the subsystem, and command factory methods
that
create commands that interact with the hardware.

A subsystem in AdvantageKit also typically won't contain any properties for the sensor values, instead, it will contain
a public instance of an Inputs object that is updated by the IO layer, and can be read from outside the subsystem.

Here's an example of a subsystem that controls a drivetrain using an IO layer:

```kotlin
/*
Assume we have a DrivetrainIO interface that has methods for controlling the drivetrain, and an Inputs object that
contains the sensor values for the drivetrain
 */

class DriveSubsystem(
    val io: DrivetrainIO
) : SubsystemBase {
    val inputs = DrivetrainIO.DrivetrainIOInputs()
    
    fun getDriveCommand(speed: Double, rotation: Double): Command {
        return this.run {
            io.drive(speed, rotation)
        }
    }
    
    fun driveDistance(distance: Double): Command {
        return this.run {
            io.drive(0.5, 0.0)
        }.until { inputs.leftWheelDistance >= distance }
    }
    
    override fun periodic() {
        io.updateInputs(inputs)
        Logger.processInputs(inputs)
    }
}
```

In this example, the `DriveSubsystem` class contains an instance of a `DrivetrainIO` interface, and an instance of a
`DrivetrainIOInputs` object that contains the sensor values for the drivetrain. The `DriveSubsystem` class also contains
methods for creating commands that interact with the hardware, and a `periodic` method that updates the sensor values
from the IO layer and logs them with the `processInputs` method of the `Logger` class.

As you can also see, when creating the subsystem you'll need to pass in an instance of the IO layer that the subsystem
will use. This allows you to easily swap out the implementation of the hardware without changing the code that uses it,
and makes it easy to test the subsystem in isolation with a simulated implementation of the IO layer.

## Conclusion
