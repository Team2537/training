# Device I/O

In this section, we will cover the basics of device I/O in WPILib. This includes how to interact with motors, sensors,
and other devices on the robot.

## Types of Communication

There are a few different ways that devices can communicate with the RoboRIO. The most common ones are:

- DIO (Digital Input/Output)
- PWM (Pulse Width Modulation)
- CAN (Controller Area Network)

### DIO (Digital Input/Output)

DIO communicates with devices using binary signals (on/off). This is useful for things like limit switches, buttons, and
simple sensors.

### PWM (Pulse Width Modulation)

PWM is a way to control the speed of motors and other devices by varying the width of a pulse. This is useful for
controlling things like simple motors, servos, and picking up signals form some forms of encoders.

The "width" of the pulse refers to the amount of time the signal is high (on) compared to the total period of the
signal.
For example, a 50% duty cycle means the signal is high for half the time and low for the other half.

### CAN (Controller Area Network)

CAN is a more advanced communication protocol that allows devices to communicate with each other over a network. This is
useful for more complex systems that require high-speed communication between devices. Each device on the network has a
unique ID that allows it to be addressed individually.

## Inputs

### DI (Digital Input)

The most basic form of input is a digital input. This is a simple on/off signal that can be used to detect things like
limit switches, buttons, and other binary signals.
Most DI devices will be connected to the RoboRIO's DIO ports. To interact with a DI device, you can use
the `DigitalInput` class. Here's an example:

```kotlin
val limitSwitch = DigitalInput(0) // Connected to DIO port labeled 0 on the RoboRIO
val isPressed = limitSwitch.get()
```

### Encoders

An encoder is a device that measures the rotation of a shaft. This is useful for measuring distances, velocities, and
angles. Encoders can be connected to the RoboRIO's DIO ports, but usually the ones we use are integrated into the motor
itself. We'll talk about motors later in this section. But for now, lets go over the different types of encoders we use:

| Name                     | Type     | Description                                                                                             |
|--------------------------|----------|---------------------------------------------------------------------------------------------------------|
| NEO Encoder              | Relative | Integrated into NEO motors. Sends signals over CAN                                                      |
| REV Through Bore Encoder | Absolute | Placed around the output shaft of a mechanism. Sends signals using Duty Cycle / PWM                     |
| Kraken / Falcon Encoder  | Relative | Integrated into Kraken and Falcon motors. Sends signals over CAN                                        |
| CTRE CANcoder            | Absolute | Uses a magnet placed inside of a shaft to determine the absolute position of it. Sends signals over CAN |

The most important one to understand is the NEO Encoder, as it is the most common one we use. The NEO Encoder is
integrated into NEO motors and sends signals over CAN. This means that you can interact with it through a CANSparkMax,
we'll go over how to create them later, but once created this is how you utilize the encoder

```kotlin
val neoMotor = CANSparkMax(0, MotorType.kBrushless)

val encoder = neoMotor.encoder // Get the encoder from the motor

val position = encoder.position // Get the current position of the motor
val velocity = encoder.velocity // Get the current velocity of the motor
```

Remember that the NEO Encoder is relative, meaning that it measures the position of the motor relative to where it was
when it was turned on (or last reset). This is different from absolute encoders, which measure the position of the motor
relative to a known starting point, that persists even when the motor is turned off.

### Gyroscopes

A gyroscope is a device that measures the orientation of an object. This is useful for things like driving straight,
turning, and keeping track of the robot's position on the field. There are a TON of options for gyroscopes, but the most
common one we use is called the Pigeon2, so that's what I'll show you how to use.

The pigeon uses a system called "StatusSignals" which is very complex so we won't go too deeply into it, but you can
read about it [HERE](https://v6.docs.ctr-electronics.com/en/stable/docs/api-reference/api-usage/status-signals.html).

That being said, here's how you would use the pigeon:

```kotlin
val pigeon = Pigeon2(0) // Device has a CAN ID of 0

val angleSignal = pigeon.getYaw() // StatusSignal that represents the yaw (heading) of the robot
val angle = angleSignal.value // Get the actual value of the signal

// You can also do it all in one line, although that can have some performance implications
val angle = pigeon.getYaw().value
```

## Outputs

### Motors

Motors are devices that convert electrical energy into mechanical energy. This is what makes the robot move! There are
many different types of motors, but the most common ones we use are NEOs and Krakens. Krakens are a bit more complex
so we won't go over them here, and instead we'll focus on NEOs.

NEOs are brushless motors that are controlled over CAN. This means that you can interact with them through a
CANSparkMax object in your code like we saw in the encoder example. Here's how you would create and use a NEO motor:

```kotlin
val neoMotor = CANSparkMax(0, MotorType.kBrushless) // Create a new NEO motor on CAN ID 0
```

Super simple! You have lots of options for controlling the motor, but the most common ones are `set` and `setVoltage`.
`set` sets the speed of the motor as a percentage of its maximum speed, while `setVoltage` sets the voltage of the motor
directly, and is therefore usually more repeatable and accurate. Here's how you would use them:

```kotlin
neoMotor.set(0.5) // Set the motor to 50% speed

neoMotor.setVoltage(6.0) // Set the motor to 6 volts
```

### Solenoids / Pistons

Solenoids use pressurized air to create linear motion. We tend not to use them too much, but they are useful for things
like quick actuation of mechanisms. Solenoids are controlled using the `Solenoid` or `DoubleSolenoid` classes.

The `Solenoid` class is used for single-acting solenoids where applying pressure makes them extend, and when you remove
pressure a spring retracts them. The `DoubleSolenoid` class is used for double-acting solenoids where applying pressure
makes them extend and applying pressure to the other port makes them retract, these do not have springs.

Here's how you would use them:

#### Single-Acting Solenoid
```kotlin
val solenoid = Solenoid(PneumaticsModuleType.REVPH, 0) // Create a new solenoid on the REV Pneumatics Hub module, with channel 0

solenoid.set(true) // Extend the solenoid
solenoid.set(false) // Retract the solenoid
```

#### Double-Acting Solenoid
```kotlin
val solenoid = DoubleSolenoid(PneumaticsModuleType.REVPH, 0, 1) // Create a new double solenoid on the REV Pneumatics Hub module with a forward channel of 0 and a reverse channel of 1

solenoid.set(DoubleSolenoid.Value.kForward) // Extend the solenoid
solenoid.set(DoubleSolenoid.Value.kReverse) // Retract the solenoid
solenoid.set(DoubleSolenoid.Value.kOff) // Turn off the solenoid

// You can also use the `toggle` method to toggle the solenoid between forward and reverse
solenoid.toggle()

// And you can use the `get` method to get the current state of the solenoid
val state = solenoid.get()

// The state will be one of the following:
// - DoubleSolenoid.Value.kForward
// - DoubleSolenoid.Value.kReverse
// - DoubleSolenoid.Value.kOff
```

## Controllers

Controllers are devices that allow you to interact with the robot. This includes things like joysticks, gamepads, and
other input devices. Controllers are connected to the computer running the driver station software, and the driver
station software sends the input from the controllers to the robot over a network connection.

To interact with controllers in your code, you can use the `Joystick` or `XboxController` classes. Here's an example of
how you would use them:

```kotlin
val joystick = Joystick(0) // Create a new joystick on USB port 0

val xAxis = joystick.getX() // Get the x-axis value of the joystick
val yAxis = joystick.getY() // Get the y-axis value of the joystick

val button1 = joystick.getRawButton(1) // Get the state of button 1 on the joystick
```

```kotlin
val controller = XboxController(0) // Create a new Xbox controller on USB port 0

val leftX = controller.leftX // Get the x-axis value of the left stick
val leftY = controller.leftY // Get the y-axis value of the left stick

val rightX = controller.rightX // Get the x-axis value of the right stick
val rightY = controller.rightY // Get the y-axis value of the right stick

val aButton = controller.aButton // Get the state of the A button on the controller
```

Controllers are a great way to interact with the robot and control its behavior. You can use them to drive the robot,
control mechanisms, and trigger actions based on user input.



## Conclusion

That's it for device I/O! You should now have a basic understanding of how to interact with motors, sensors, and other
devices on the robot. If you have any questions, check the following resources:
- [WPILib Hardware API Docs](https://docs.wpilib.org/en/latest/docs/software/hardware-apis/index.html)
- [CTRE Phoenix API Docs](https://v6.docs.ctr-electronics.com/en/stable/index.html)
- [REV Robotics API Docs](https://docs.revrobotics.com/brushless/spark-flex/revlib)

