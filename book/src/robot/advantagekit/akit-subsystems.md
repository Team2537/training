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

## Logging Outputs

When creating a subsystem, you wont just want to log the inputs into your control logic, you'll also want to log the
outputs of the subsystem. This is useful for debugging, and for tuning the control loops of the subsystem.

Some outputs could be the estimated pose of odometry, the estimated pose from vision, or the trajectory that the robot is
following. These outputs can be logged easily using the `Logger.recordOutput` method, which takes a key and a value. This
value can be a number, a boolean, a string, or a complex object that implements `WPISerializable`, such as a Pose2d or
another geometry object.

The `Logger.recordOutput` method is called in the `periodic` method of the subsystem, as it must be called every loop
iteration to log the outputs. Here's an example of how you could log the estimated pose of odometry in the `DriveSubsystem`:

```kotlin
class DriveSubsystem(
    val io: DrivetrainIO
) : SubsystemBase {
    val inputs = DrivetrainIO.DrivetrainIOInputs()
    val odometry = Odometry() // Not how odometry is actually implemented, but you get the idea
    
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
        
        Logger.recordOutput("Estimated Pose", Pose2d.struct, odometry.estimatedPose)
    }
}
```

In this example, the `DriveSubsystem` class contains an instance of an `Odometry` class that estimates the pose of the
robot, and the `periodic` method of the `DriveSubsystem` class logs the estimated pose of the robot using the
`Logger.recordOutput` method.

You may also notice that we passed more than just a key and a value to the `Logger.recordOutput` method. The `Pose2d`
class is a complex object that implements `WPISerializable`, and is used to serialize the `Pose2d` object into a format
that can be logged. This is useful for logging complex objects that can't be logged directly, such as geometry objects or
other complex objects. To do this you need to pass in the `struct` property of the complex object. This tells the logger
to serialize the object into a format that can be logged.

### Logging Arrays
Collections (Arrays, Lists, ETC) can also be logged using the `Logger.recordOutput` method. This is useful for logging
arrays of sensor values, or other collections of values. To log an array, you do the same thing as logging any other
value, but rather than just passing in the nane of the value you want to log, you add the `*` operator before the array
name. This "unwraps" the array so AdvantageKit can process it properly.

Here's an example of how you could log an array of vision target poses in the `DriveSubsystem`:

```kotlin
class VisionSubsystem(
    val io: VisionIO
) : SubsystemBase {
    val inputs = VisionIO.VisionIOInputs()
    
    override fun periodic() {
        io.updateInputs(inputs)
        Logger.processInputs(inputs)
        
        Logger.recordOutput("Vision Targets", Pose2d.struct, *inputs.result.visionTargets)
    }
}
```

In this example, the `VisionSubsystem` class contains an instance of a `VisionIO` interface, and an instance of a
`VisionIOInputs` object that contains the sensor values for the vision system. The `VisionSubsystem` class logs an array
of vision target poses using the `Logger.recordOutput` method.

## Conclusion

AdvantageKit is a very powerful library that allows you to easily test different combinations of hardware on your robot. The ability to create a subsystem decoupled from its hardware is incredibly powerful for things like simulation and logging in a very controlled manner.

The AdvantageKit GitHub repository has a ton of example projects and resources if you would like to look more into this, you can find this [HERE](https://github.com/Mechanical-Advantage/AdvantageKit)