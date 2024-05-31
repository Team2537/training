# PID Control

PID control is a type of control system that uses feedback from sensors to control a mechanism. It's used in many
different applications, from controlling the speed of a motor to controlling the position of an arm. In this section,
we'll learn how to use PID control in FRC robot code.

## What is PID Control?

PID control is a control system that uses feedback from sensors to control a mechanism. It stands for Proportional,
Integral, Derivative, which are the three components of the control system.

Each component adjusts the output of the control system based on the "error", which is the difference between the
desired
value (setpoint) and the actual value. The three components work together to adjust the output in different ways:

- **Proportional (P)**: Adjusts the output based on the current error. The larger the error, the larger the output.
- **Integral (I)**: Adjusts the output based on the sum of the errors over time. This helps eliminate steady-state
  error.
- **Derivative (D)**: Adjusts the output based on the rate of change of the error. This helps reduce overshoot and
  oscillations.

Those of you who have taken calculus may recognize these terms. Integral and Derivative are essentially the same as the
integral and derivative in calculus. Integral is the area under the curve, and Derivative is the slope of the curve at a
point.

## How Does PID Control Work?

PID is a type of "closed-loop" control system, which means it uses feedback from sensors to adjust the output. The
control system works by comparing the desired value (setpoint) to the actual value (measured by sensors) and adjusting
the output based on the error.

The control system calculates the error by subtracting the setpoint from the actual value. It then calculates the
output by summing the three components (P, I, D) multiplied by their respective constants (Kp, Ki, Kd). The output is
then sent to the mechanism to control it.

Those constants (Kp, Ki, Kd) are called "tuning constants" and are used to adjust the behavior of the control system.
Tuning the PID constants can be a complex process and often requires trial and error to get right.

## Implementing PID Control in FRC

In FRC we have two main ways to implement PID control: using the `PIDController` class in WPILib, or utilizing vendor
libraries to run PID control on the motor controllers themselves. Running PID control on the motor controllers is
preferred as it offloads the computation from the RoboRIO to the motor controller, which can be more efficient, and it
can run at a higher frequency.

### Using the `PIDController` Class

The `PIDController` class in WPILib is a simple way to implement PID control in your robot code. It handles the
calculation of the PID output. but it's your responsibility to use that output to control the mechanism, and make sure
you're repeatedly calling the `calculate()` method.

Here's an example of how to use the `PIDController` class to control the speed of a motor:

```kotlin
val motor = CANSparkMax(1, MotorType.kBrushless)
val encoder = motor.encoder

val pidController = PIDController(0.1, 0.01, 0.0) // Kp, Ki, Kd

motor.set(
    pidController.calculate(
        encoder.position,
        100.0
    )
) // Calculate with a measured value from the encoder, and a setpoint of 100.0

motor.set(pidController.calculate(encoder.position)) // Once you've set the setpoint, you can just call calculate with the measured value, and it will use the setpoint you've set before
```

### Using Vendor Libraries

Most modern motor controllers have built-in PID control that you can use. Since it's your first year, and krakens are
complex, we'll focus on the SparkMax, which has built-in PID control using the `SparkPIDController` class.

Here's an example of how to use the `SparkPIDController` class to control the speed of a motor:

```kotlin
val motor = CANSparkMax(1, MotorType.kBrushless)
val pidController = motor.pidController

pidController.p = 0.1
pidController.i = 0.01
pidController.d = 0.0

pidController.setReference(100.0, ControlType.kVelocity) // Set the setpoint to 100 rpm
```

You might notice that we don't have to call `calculate()` like we did with the `PIDController` class. That's because the
`SparkPIDController` class handles the calculation and control of the motor for you.

You may also notice that we set the control type to `kVelocity`. This tells the motor controller that we're attempting
to control the velocity of the motor. This causes it to send the velocity readings from the encoder into the control
loop as your actual value. You can also set it to `kPosition` to control the position of the motor, as well as several
other options that we will not cover here.

The SparkMax PID controller will operate based off of the conversion factors you set in the motor controller, so make
sure you set those correctly to convert RPM to feet per second or whatever units you want to set velocity in.

## Summary

PID control is a powerful tool for controlling mechanisms on your robot. It uses feedback from sensors to adjust the
output based on the error between the desired value and the actual value. In FRC, you can implement PID control using
the `PIDController` class in WPILib or using vendor libraries like the `SparkPIDController` class in the SparkMax
library. Running PID control on the motor controllers is preferred as it offloads the computation from the RoboRIO to
the motor controller, which can be more efficient.

## Additional Resources
These ones are incredibly important, if you do any reading from these sections, make sure it's these ones.

- [WPILib Documentation - PIDController](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/controllers/pidcontroller.html)
- [Matlab - PID Control](https://www.mathworks.com/videos/series/understanding-pid-control.html) Video 1 is the most important, but the whole series is good.