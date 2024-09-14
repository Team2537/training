# Robot Code & Robot Structure

Up until this point we've only really talked about controlling individual parts of a robot, but that isn't that useful
is it? Luckily WPILIB provides a system of structuring your hardware control code that makes it easy to control larger
parts of the robot like mechanisms, and more complex systems like a computer vision pipeline.

This system is called "Command Based Programming" and it's a way of organizing your code into subsystems and commands
that
interact with each other. This makes it easy to write code that controls the robot in a way that makes sense, and is
easy
to understand. It also provides some aspects of safety that we'll talk about later.

## Suppliers

Although not directly related to Command Based Programming, it's important to understand the concept of Suppliers.
When you pass a `double` to a method, you pass in that value at that time, this is fine for a lot of situations, but
what about joysticks? Joysticks are constantly changing, so you need a way to get the value of the joystick every time
you need it, without calling the method again, like in a loop within a command.

Suppliers store an action that can be called to get a value, and when you call the `get()` method on the supplier, it
calls that action and returns the value. This is useful for things like joysticks, where you want to get the value of
the
joystick every time a loop runs within a function, rather than running that function in a loop.

Suppliers are super easy to declare, all you really need right now is double suppliers, which are suppliers that return
a double. Here's an example of a supplier that returns the value of a joystick axis, DoubleSuppliers are unique in that
they have a property called `asDouble` that returns the value of the supplier, rather than calling the `get()` method.

```kotlin
val joystick = Joystick(0)
val axisSupplier = DoubleSupplier { joystick.getRawAxis(0) }

println(axisSupplier.asDouble) // Prints the value of the joystick axis
```

## Subsystems

The first part of Command Based Programming is Subsystems. Subsystems are classes that represent a part of the robot,
like the drivetrain, or an arm mechanism. They contain the hardware objects that control that part of the robot, like
motors, sensors, and solenoids.

Subsystems are intended to be used to store information (state) of a subsystem, as well as control access to the hardware to make sure that it is always only being given one command, and always being controlled in specific "allowed" ways.

Subsystems typically contain three parts

- Properties: These are the hardware objects that control the subsystem. For example, a drivetrain subsystem might have
  two motors, one for the left side and one for the right side.
- Control Methods: These are the functions that control the subsystem. For example, a drivetrain subsystem might have a
  method
  called `drive` that takes a speed and a rotation and drives the robot at that speed and rotation.
- Command Factory Methods: These are methods that create commands that control the subsystem. For example, a drivetrain
  subsystem might have a method called `createDriveCommand` that creates a command that drives the robot at a certain
  speed and rotation.

Here's an example of a simple subsystem that controls a drivetrain

```kotlin
import java.util.function.DoubleSupplier

class Drivetrain : Subsystem() {
    private val leftMotor = CANSparkMax(0, MotorType.kBrushless)
    private val rightMotor = CANSparkMax(1, MotorType.kBrushless)

    fun drive(speed: Double, rotation: Double) {
        leftMotor.set(speed + rotation)
        rightMotor.set(speed - rotation)
    }

    fun createDriveCommand(speed: DoubleSupplier, rotation: DoubleSupplier): Command {
        // Note the use of suppliers, as whatever you place within the run method will be run every robot loop until its cancelled
        // We'll discuss this more later, just make sure you note the use of suppliers
        return this.run {
            drive(speed.asDouble, rotation.asDouble)
        }
    }
}
```

## Commands

Commands are classes that control a subsystem. They contain the logic that controls the subsystem, like driving the
robot, or moving an arm mechanism. The advantage of commands is that they stop you from controlling the subsystem in
multiple places at once, which can cause bugs, or slamming the arm mechanism into the air filtration units.

Commands can be created in many ways, but as a rookie the important ones to know are the Subsystem factory methods,
which are contained in the table below.

| Method     | Description                                                                                    |
|------------|------------------------------------------------------------------------------------------------|
| `run`      | Runs a lambda every robot loop until the command is cancelled                                  |
| `runEnd`   | Runs one lambda every loop until command is cancelled, runs a second one one time when it ends |
| `runOnce`  | Runs a lambda one time when the command is started                                             |
| `startEnd` | Runs a lambda one time when the command is started, runs a second one one time when it ends    |

By lambdas, we mean the code that is placed within the brackets of the method. For example, in the `run` method, the
lambda is `drive(speed.asDouble, rotation.asDouble)`. This lambda will be run every robot loop until the command is
cancelled.

To create a command factory method, you want to create a method and use the factory methods shown above to create a
command. running it on `this` will make sure that the command is associated with the subsystem, meaning no other
commands can run on the subsystem at the same time.

Here's an example of a command that drives a robot at a certain speed and rotation

```kotlin
fun createDriveCommand(speed: DoubleSupplier, rotation: DoubleSupplier): Command {
    return this.run {
        drive(speed.asDouble, rotation.asDouble)
    }
}
```

## Command Groups

Command Groups are classes that contain multiple commands that run in sequence. This is useful for complex actions
that require multiple steps, like picking up a game piece, driving to a location, and scoring the game piece.

Command Groups are easiest to create using the method chaining API, which is a way of creating a command group by
chaining methods together. This is done different ways depending on the type of command group, but the most common
ones are sequential groups using `.andThen()` and parallel groups using `.alongWith()`.

Here's an example of a command group that drives the robot forward for 3 seconds, then stops, turns 90 degrees, and
drives forward for 3 more seconds

```kotlin
val driveForward = Drivetrain.createDriveCommand({ 0.5 }, { 0.0 }).withTimeout(3.0)
val turn = Drivetrain.createDriveCommand({ 0.0 }, { 0.5 }).withTimeout(1.0) // Assume 1s is 90 degrees

val group = driveForward.andThen(turn).andThen(driveForward)
```

## Scheduling Commands & Triggers

Remember suppliers? They're back! This time, they're used to schedule commands. Triggers are basically fancy boolean
suppliers that allow you to tie commands to certain conditions. For example, you could tie a command to a button press,
or a button release, or anything else that can be represented with a boolean value!

Here's an example of a trigger that runs a command when a button is pressed

```kotlin
val joystick = Joystick(0)

val trigger = Trigger { joystick.getRawButton(1) }
val command = Drivetrain.createDriveCommand({ 0.5 }, { 0.0 }).withTimeout(3.0)

trigger.onTrue(command)
```

This code will run the command when button 1 on joystick 0 is pressed until 3 seconds have passed. Although this is kind
of verbose isn't it? You can actually simplify this code to just use the `CommandJoystick` (and `CommandXboxController`)
class, which is a joystick that has methods that return triggers directly, rather than booleans that you have to wrap in
a trigger.

```kotlin
val joystick = CommandJoystick(0)

val command = Drivetrain.createDriveCommand({ 0.5 }, { 0.0 }).withTimeout(3.0)

joystick.button(1).onTrue(command)
```

This code does the same thing as the previous code, but it's a lot simpler and easier to understand.

Finally, you may notice that we've been using `onTrue` to run a command when a trigger is true, but what if you want to
run a command based on other conditions?
The [Trigger JavaDocs](https://github.wpilib.org/allwpilib/docs/release/java/edu/wpi/first/wpilibj2/command/button/Trigger.html)
contain information on all the different methods you can use to run commands based on different conditions, but the
other two important ones are `whileTrue` which runs a command while a trigger is true but cancels it if its no longer
true, and `toggleOnTrue` which runs a command when a trigger is true, but cancels it if the trigger is true again.

## The Robot Class

Now that we've talked about commands, and triggers, lets talk about how we use them.

The `Robot` class is the main class of your robot code. It's responsible for initializing the subsystems and commands,
as well as running the "Command Scheduler" which is the part of WPILIB that actually tracks and runs commands.

The `Robot` class inherits from `TimedRobot`, this gets it a `init` and `periodic` method for each of the robot modes (
disabled, autonomous, teleop, etc) as well as an overall `robotPeriodic` method that runs every robot loop, regardless
of the mode. All periodic methods are run every 20ms, or 50 times a second, this is our "loop time".

You may notice the lack of a `robotInit` method, this is because the constructor is just as good for initializing the
robot, and there's no reason to not just declare all of your commands within the `Robot` object.

### Binding Commands in the Robot Class

The `Robot` class is where you bind commands to triggers. This is done by creating triggers and commands in the
`Robot` class, and then binding them together using the `onTrue`, `whileTrue`, and `toggleOnTrue` methods as discussed
earlier.

You can also bind commands to triggers in the `Robot` class using the `CommandJoystick` and `CommandXboxController`,
extensions of the `Joystick` and `XboxController` classes that have methods that return triggers directly, instead of
booleans that you have to wrap in a trigger. This can be done cleanly using the `apply` method like when initializing
hardware objects.

Here's an example of binding a command to a button press in the `Robot` class

```kotlin
class Robot : TimedRobot() {
    private val controller = CommandXboxController(0).apply {
        b().onTrue(PrintCommand("Hello, World!"))
        b().onFalse(PrintCommand("Goodbye, World!"))
    }
}
```

as you can see, we've created a `PrintCommand` that prints "Hello, World!" when a button is pressed, and "Goodbye,
World!"
when the button is released. We then bind the command to the button press using the `onTrue` method.

### Setting default commands

Often you will want your subsystems to have a default command that runs when no other command is running. This is done
by setting the `defaultCommand` property of the subsystem to the command you want to run. Once again, this is done in
the `Robot` class using the `apply` method.

Here's an example of setting a default command for a subsystem in the `Robot` class

```kotlin
class Robot : TimedRobot() {
    private val drivetrain = Drivetrain().apply {
        defaultCommand = Drivetrain.createDriveCommand({ 0.0 }, { 0.0 })
    }
}
```

## Conclusion

Command Based Programming is a powerful way to structure your robot code. It makes it easy to write code that is
organized, easy to understand, and safe. It also provides a way to control complex systems like mechanisms and computer
vision pipelines.

We've only scratched the surface of Command Based Programming in this module. There's a lot more to learn, but this
should give you a good starting point. If you want to learn more (and you should), you can check out the WPILIB docs as
always. Some good starting points are the following

- [WPILIB Command Based Programming Docs](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html)