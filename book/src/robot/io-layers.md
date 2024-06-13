# IO Layers

IO Layers are a fundamental concept when using the `AdvantageKit` library. `AdvantageKit` is primarily used to provide
abstract interfaces for controlling groups of hardware, which can then be used to easily log and simulate said hardware.

## What is an IO Layer?

An IO Layer is a class that exposes a set of methods for interacting with a specific piece or group of hardware. For
example, a `Gripper` IO Layer might expose methods for opening and closing the gripper, while a `DriveTrain` IO Layer
might
expose methods for driving the robot.

IO Layers are designed to be simple and easy to use, and are intended to abstract away the complexities of the hardware
they control. This makes it easy to write code that interacts with the hardware, without having to worry about the
specifics of how the hardware works.

## Creating an IO Layer

Creating an IO Layer involves two steps, creating the IO Layer interface, which is basically a list of methods that the
IO Layer will expose, and than as many IO Layer implementations as you need, which are the classes that actually
implement
the methods, but for different sets of hardware, or on a more basic level, real vs simulated hardware.

The interface also contains a nested `Inputs` class, which contains a list of values that the IO Layer will read from
the hardware. These values then get logged and can be used for match replays, and similar things.

### Creating the Interface

The interface is a simple Kotlin interface, with a nested `Inputs` class. Here is an example of a simple IO Layer
interface for a `Gripper`:

```kotlin
interface GripperIO { // Define the interface for the Gripper
    class GripperIOInputs : LoggableInputs { // Define the inputs class
        var isOpen: Boolean = false // Define an input for the gripper state
        var holdingCube: Boolean = false // Define an input for whether the gripper is holding a cube
        var cubeDistance: Double = 0.0 // Define an input for the distance of the cube from the back of the gripper

        override fun toLog(table: LogTable?) {
            table?.put("isOpen", isOpen)
            table?.put("holdingCube", holdingCube)
            table?.put("cubeDistance", cubeDistance)
        }

        override fun fromLog(table: LogTable?) {
            isOpen = table?.get("isOpen") ?: isOpen
            holdingCube = table?.get("holdingCube") ?: holdingCube
            cubeDistance = table?.get("cubeDistance") ?: cubeDistance
        }
    }

    fun updateInputs(inputs: GripperIOInputs) // Define a method for updating the inputs of the gripper
    fun openGripper() // Define a method for opening the gripper
    fun closeGripper() // Define a method for closing the gripper
}
```

As you can see, it's a very simple concept. You may also notice the `LoggableInputs` interface, which is a simple
interface that defines methods for converting the inputs to and from a `LogTable`, which is a simple key-value store
that is used for logging.

### Creating the Implementation

Now that you've defined the interface, you need to create an implementation of the interface. Here is an example of a
simple implementation of the `GripperIO` interface using solenoids:

```kotlin
class GripperIOSolenoids : GripperIO {
    val solenoid = DoubleSolenoid(0) // Create a new Solenoid on port 0
    val sensor = Ultrasonic(0, 1) // Create a new Ultrasonic sensor on ports 0 and 1

    override fun updateInputs(inputs: GripperIOInputs) {
        inputs.isOpen = solenoid.get() == Value.kForward
        inputs.holdingCube = sensor.getRangeMM < 100
        inputs.cubeDistance = sensor.getRangeInches
    }

    override fun openGripper() {
        solenoid.set(Value.kForward)
    }

    override fun closeGripper() {
        solenoid.set(Value.kReverse)
    }

}
```

This implementation uses a `DoubleSolenoid` to control the gripper, and an `Ultrasonic` sensor to detect if a cube is
present in the gripper. The `updateInputs` method reads the state of the solenoid and the distance of the cube from the
back of the gripper, and updates the inputs accordingly.

You may notice that we're not exposing any methods for reading the inputs, this is because when using an IO layer, the
inputs will be created elsewhere, and when the `updateInputs` method is called, it will update the inputs with the
current state of the hardware, so then you can read the inputs from the inputs object.

### Creating Inputs out of Measures (Units Library)

WPILib's java units library doesn't translate perfectly to AdvantageKit's inputs like doubles do, so let's real quick
go over how you create your toLog and fromLog methods when using Measures.

```kotlin
interface FlywheelIO {
    class FlywheelIOInputs : LoggableInputs {
        var speed: MutableMeasure<Velocity<Angle>> = MutableMeasure.zero(Units.RadiansPerSecond)
        var voltage: MutableMeasure<Voltage> = MutableMeasure.zero(Units.Volts)

        // Luckily, you can just put the measure directly into the table
        override fun toLog(table: LogTable?) {
            table?.put("speed", speed) 
            table?.put("voltage", voltage)
        }
        
        // This is where it gets a bit more complicated
        override fun fromLog(table: LogTable?) {
            speed.mut_replace(table?.get("speed", speed)) // Replace the speed measure with the one from the table, or keep the old one if it can't be found
            voltage.mut_replace(table?.get("voltage", voltage)) // Replace the voltage measure with the one from the table, or keep the old one if it can't be found
        }
    }
    
    fun updateInputs(inputs: FlywheelIOInputs)
    fun setVoltage(voltage: Measure<Voltage>)
}
```

Your updateInputs also looks slightly different, so lets quickly go over that as well for completeness.

```kotlin
class FlywheelIOSpeedController : FlywheelIO {
    val motor = CANSparkMax(0, MotorType.kBrushless) // Create a new SparkMax on port 0
    
    override fun updateInputs(inputs: FlywheelIOInputs) {
        inputs.speed.mut_replace(motor.encoder.velocity, Units.RPM) // Assuming the encoder is outputting RPM velocity readings
        inputs.voltage.mut_replace(motor.appliedOutput * motor.busVoltage, Units.Volts)
    }
    
    override fun setVoltage(voltage: Measure<Voltage>) {
        motor.setVoltage(voltage.value)
    }
}
```

## Conclusion

IO Layers are a powerful tool for abstracting away the complexities of hardware, and making it easy to write code that
interacts with the hardware. By creating simple interfaces and implementations, you can easily control and log hardware
in your robot code, without having to worry about the specifics of how the hardware works. This makes it easy to write
clean, maintainable code that is easy to understand and debug.

## Additional Resources

- [AdvantageKit Documentation](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/docs/RECORDING-INPUTS.md)