# Open-Loop Control

The last two modules in this unit cover the basics of "Control Systems" in robotics. Control systems are the algorithms
that determine how a robot behaves in response to requested actions and sensor input. This first module covers
"Open-Loop Control", which will be explained in the rest of the module.

## What is Open-Loop Control?

As stated before, Control systems are the algorithms that determine how a robot behaves in response to requested actions
and in some cases sensor input.

Open-Loop control systems are the simplest form of control systems. They are called "Open-Loop" because they do not
actually utilize any sensor feedback to update the requested actions. Instead, the robot simply executes the requested
actions without any regard to the environment or the robot's current state, assuming that the robot will always behave
the same way given the same requested actions.

This is a very simple and straightforward way to control a robot, but it is also very limited. Open-Loop control systems
are not able to adapt to changing conditions or unexpected events, and are generally not very reliable. However, they
are useful in many cases in FRC, where precise control is not necessary.

### Duty Cycle / Voltage Control

These are the simplest forms of Open-Loop control. They simply invole telling a motor to run at a certain percentage of
its maximum speed, or to run at a certain voltage. This is the most basic form of control, but they come with several
drawbacks.

#### Duty Cycle

Duty Cycle control involves telling a motor to run at a certain percentage of its maximum speed. The downside to this is
that the actual speed of the motor can vary greatly depending on the load on the motor. For example, if the motor is
under a heavy load, it may not be able to reach the requested speed, and if the motor is under a light load, it may run
faster than requested.

#### Voltage Control

Voltage control is similar to Duty Cycle control, but instead of telling the motor to run at a certain percentage of its
maximum speed, you tell it to run at a certain voltage. This is slightly more accurate than Duty Cycle control, and is
much more repeatable in terms of consistent speeds. The major downside of this is that that concistency can be thrown
off by the battery voltage, which can vary greatly depending on the charge of the battery.

### Implemation Example

Voltage and Duty Cycle control can be utilized with SparkMax motor controllers using `.set()` and `.setVoltage()`
methods.

```kotlin
val motor = CANSparkMax(1, MotorType.kBrushless)
motor.set(0.5) // Run the motor at 50% speed
motor.setVoltage(6.0) // Run the motor at 6 volts
```

Very simple, right? These methods are very easy to use, but they are not very reliable. They are useful for simple tasks
where precision is not necessary, but they are not recommended for more complex tasks.

### Feedforward Control

Feedforward control is a much more advanced form of Open-Loop control, and is in fact the main form of control used for
a lot of systems in FRC. Feedforward control is really a fancy name for a function that takes in a velocity, and returns
a voltage that will make the motor run at that velocity.

You may be wondering "How is this different from Voltage Control?" The difference is that Feedforward control takes into
account the characteristics of the motor and the system it is controlling, and can adjust the voltage based on that. For
example, the feedforward functions take into account the amount of voltage required to overcome the static friction of
the motor.

#### The Math

The basic formula for Feedforward control is as follows: \\( V(x) = ( K_v * x) + ( K_a * \dot x ) + ( K_s * sign(
x) ) \\)

Now, let's break down what each of these terms mean:

- \\( V(x) \\) is the voltage that the motor should run at to achieve the desired velocity \\( x \\).
- \\( K_v \\) is the velocity gain, which is the amount of voltage required to make the motor run at a certain velocity,
  and is usually measured in volts per meter per second.
- \\( K_a \\) is the acceleration gain, which is the amount of voltage required to make the motor accelerate at a
  certain
  rate, and is usually measured in volts per meter per second squared.
- \\( K_s \\) is the static friction gain, which is the amount of voltage required to overcome the static friction of
  the
  motor, and is usually measured in volts.
- \\( x \\) is the desired velocity of the motor as stated before.
- \\( \dot x \\) is the desired acceleration of the motor.
- \\( sign(x) \\) is the sign of the velocity, which is used to determine the direction of the motor, and therefore the
  direction of the voltage.

This formula is a very simple equation on the surface, but it can be very powerful when used correctly. It allows you to
control the motor in a very precise way, and can be used to make the motor behave in a very specific way.

#### Downside of Feedforward Control

Like all Open-Loop control systems, Feedforward control is not perfect. It is not able to adapt to changing conditions
or
correct for errors in the system. This means that if the system is not tuned correctly, the motor may not behave as
expected, and may not be able to achieve the desired velocity.

#### How to find the gains

For simple motor feedforward, it's very easy to find K_v, and K_s. K_a is a lot harder to find, but luckily, it's not
usually necessary to tune it.

To find your K_v and K_s you use the following steps:

- Run the motor at several different voltages, and measure the velocity of the motor at each voltage.
- Plot the voltage as a function of the velocity, and find the slope of the line. This is your K_v.
- Find the y-intercept of the line. This is your K_s.

### Implementation Example

WPIlib provides a `SimpleMotorFeedforward` class that can be used to implement feedforward control.
Remember, feedforward outputs a voltage, so you need to use `.setVoltage()` to set the motor to the desired voltage,
rather than `.set()`.

```kotlin
val motor = CANSparkMax(1, MotorType.kBrushless)
val feedforward = SimpleMotorFeedforward(0.1, 0.2, 0.05) // K_v, K_a, K_s

val velocity = 5.0 // Desired velocity in meters per second
val acceleration = 2.0 // Desired acceleration in meters per second squared

val voltage = feedforward.calculate(velocity, acceleration)
motor.setVoltage(voltage)
```

This is a very simple example of how to use feedforward control. It is a very powerful tool, and can be used to control
the motor in a very precise way. Feedforward can also be used to take gravity and other external forces into account,
but those will not be covered, and if you are interested, we will provide resources for you to learn more.

## Conclusion

Open-Loop control is the simplest form of control systems, and is not able to adapt to changing conditions or correct
for errors in the system. It is useful for simple tasks where precision is not necessary, but is not recommended for
more complex tasks.

Duty Cycle and Voltage control are the simplest forms of Open-Loop control, and are not very reliable. They are useful
for simple tasks where precision is not necessary, but are not recommended for more complex tasks.

Feedforward control is a much more advanced form of Open-Loop control, and is in fact the main form of control used for
a lot of systems in FRC. It is a very powerful tool, and can be used to control the motor in a very precise way.

## Additional Resources
- [WPILib Feedforward Control](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/controllers/feedforward.html)
