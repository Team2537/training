# Closed-Loop Control

Now that we've covered Open-Loop control, we can move on to Closed-Loop control. Closed-Loop control is on a basic level
pretty simple to understand, but it can get complicated (and interesting) pretty fast. The main idea behind Closed-Loop
control is that rather than output being based on purely the input, the output is based on the input, as well as the
measured distance between the output and input. This allows the system to correct for errors in the system, and adapt to
changing conditions.

## Proportional Control

Proportional control is the simplest form of Closed-Loop control, and is the most common form of Closed-Loop control
used
in FRC. Proportional control simply takes the distance (or error) between the desired value and the actual value, and
multiplies it by a constant, called the Proportional Gain or `K_p`. This value is then output to the motor.

Alternatively, rather than simply outputting that value, you can add it to the output of an Open-Loop control system,
such as Feedforward control. This is incredibly useful as pure Proportional control is good at error correction, but it
can be difficult to find a `K_p` that both quickly reaches the target, but doesn't overshoot it. By using Feedforward
control to get close to the target, and Proportional control to correct for errors, you can get the best of both worlds.

### Implementing Proportional Control

Lucky for us, WPILib has a built-in class that can be used to implement Proportional control, called `PIDController`.
This class can be used to implement other forms of Closed-Loop control as well, but we'll get to that later.

```kotlin
val K_p = 0.1
val motor = CANSparkMax(1, MotorType.kBrushless)

val controller: PIDController =
    PIDController(K_p, 0.0, 0.0) // Create a new PIDController with a K_p of 0.1, for now ignore the other values
controller.setSetpoint(100.0) // Set the desired position of the motor to 100 rotations

// Placeholder function for something getting called periodically
fun periodic() {
    val output =
        controller.calculate(motor.encoder.position) // Calculate the output of the PIDController based on the current position of the motor, and the desired position
    motor.setVoltage(output) // Set the motor to the output of the PIDController
}
```

As you can see, implementing Proportional control is pretty simple. The `PIDController` class takes care of all the
math for you, and all you need to do is call the `calculate()` function with the current position of the motor, and the
desired position. The `setVoltage()` function is used to set the motor to the output of the `PIDController`.

### Implementing Feedforward and Proportional Control

Implementing Feedforward and Proportional control is also pretty simple. All you need to do is add the output of the
`PIDController` to the output of the `SimpleMotorFeedforward` class.

```kotlin
val K_p = 0.1
val motor = CANSparkMax(1, MotorType.kBrushless)

val feedforward = SimpleMotorFeedforward(0.1, 0.2, 0.05) // K_v, K_a, K_s
val controller: PIDController =
    PIDController(K_p, 0.0, 0.0) // Create a new PIDController with a K_p of 0.1, for now ignore the other values
controller.setSetpoint(100.0) // Set the desired velocity of the motor to 100 rotations per minute

// Placeholder function for something getting called periodically
fun periodic() {
    val output =
        feedforward.calculate(controller.setpoint) + controller.calculate(motor.encoder.position) // Calculate the output of the PIDController based on the current position of the motor, and the desired position
    motor.setVoltage(output) // Set the motor to the output of the PIDController
}
```

As you can see, implementing Feedforward and Proportional control is pretty simple. All you need to do is add the output
of the `PIDController` to the output of the `SimpleMotorFeedforward` class, and set the motor to the output of that
calculation.

### Implementing Proportional Control and Feedforward Control on a SparkMax

SparkMaxs have a built-in PID controller that can be used to implement Proportional control. This is a lot simpler than
using the `PIDController` class, as the SparkMax takes care of all the math for you.

```kotlin
val K_p = 0.1
val motor = CANSparkMax(1, MotorType.kBrushless)

val controller = motor.pidController // Get the built-in PID controller of the SparkMax

controller.p = K_p // Set the Proportional gain of the PID controller to 0.1
controller.setReference(100.0, ControlType.kPosition) // Set the desired position of the motor to 100 rotations
```

As you can see, implementing Proportional control on a SparkMax is pretty simple. All you need to do is set the
Proportional
gain of the built-in PID controller, and set the desired position of the motor. Unlike with the `PIDController` class,
you only need to set the setpoint once, and the SparkMax will take care of the rest.

Feedforward control can be added to the `setReference()` function by adding some extra parameters.

```kotlin
val K_p = 0.1
val motor = CANSparkMax(1, MotorType.kBrushless)

val feedforward = SimpleMotorFeedforward(0.1, 0.2, 0.05) // K_v, K_a, K_s
val controller = motor.pidController // Get the built-in PID controller of the SparkMax

controller.p = K_p // Set the Proportional gain of the PID controller to 0.1

controller.setReference(
    100.0,
    ControlType.kVelocity,
    0, // Slot 0 (ignore this, just needed to add feedforward)
    feedforward.calculate(
        100.0,
        0.0
    ), // Calculate the feedforward output based on the desired velocity and acceleration
    ArbFFUnits.kVoltage // Set the units of the arbitrary feedforward to volts
)
```

As you can see, adding Feedforward control to the SparkMax is pretty simple. All you need to do is calculate the output
of
the `SimpleMotorFeedforward` class, and add it to the `setReference()` function. You also need to set the units of the
arbitrary feedforward to volts, as the SparkMax expects the output of the feedforward to be in volts.

Unfortunately, the SparkMax's feedforward capabilities are limited, and require you to calculate the feedforward output
on the RoboRIO, which means that we're limited in how fast we can run the feedforward loop.


#### What unit is K_p in?

The unit of `K_p` is similar to the units of the feedforward gains, where it is based on the units of the input and
output. For example, if the input is in meters, and the output is in volts, then `K_p` would be in volts per meter.
But rather than being volts per meter of the target, it is volts per meter of error (or distance from the target).

## Conclusion

In this chapter, we covered the concept of Closed-Loop control, and how it can be used to correct for errors in the
system. We specifically covered Proportional control, and how it can be used to correct for errors in the system. We also
covered how to implement Proportional control using the `PIDController` class, and how to implement Proportional control
on a SparkMax. We also covered how to implement Feedforward and Proportional control, and how to implement Feedforward
and Proportional control on a SparkMax.

## Additional Resources

The information below covers PID control, which is a more advanced form of Closed-Loop control. It is not necessary for
basic parts of our robot, but if you want to be more involved in the complex and interesting parts of our robot, it is
definitely worth looking into.

- [PID Control WPILib Docs](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/introduction/introduction-to-pid.html#introduction-to-pid)
- [Combining Feedforward and PID Control](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/controllers/combining-feedforward-feedback.html)
- [MatLab PID Video](https://www.mathworks.com/videos/understanding-pid-control-part-1-what-is-pid-control--1527089264373.html)
