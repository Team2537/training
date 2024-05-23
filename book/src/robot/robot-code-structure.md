# Robot Code & Robot Structure

Up until this point we've only really talked about controlling individual parts of a robot, but that isn't that useful
is it? Luckily WPILIB provides a system of structuring your hardware control code that makes it easy to control larger
parts of the robot like mechanisms, and more complex systems like a computer vision pipeline.

This system is called "Command Based Programming" and it's a way of organizing your code into subsystems and commands
that
interact with each other. This makes it easy to write code that controls the robot in a way that makes sense, and is
easy
to understand. It also provides some aspects of safety that we'll talk about later.

## Subsystems

Subsystems are the building blocks of Command Based Programming. They represent a part of the robot, physical or
otherwise,
that you want to control. For example, you might have a `DriveSubsystem` that controls the robot's drivetrain, or a
`ShooterSubsystem` that controls the robot's shooter.

Subsystems are made up of two parts: properties and methods. Properties are things like motors, sensors, and other
hardware
that the subsystem uses. Methods are functions that control the hardware in some way.

When creating a subsystem, you should think about what hardware it needs to control, and what methods you need to
control
that hardware. For example, a `DriveSubsystem` might have two motors and a gyro, and methods to drive the robot forward,
backward, and turn.

Creating a subsystem is pretty simple. You just need to create a new class that extends `SubsystemBase` and add the
hardware and methods you need. Here's an example of a `DriveSubsystem`:

```kotlin
class DriveSubsystem : SubsystemBase() {
    private val leftMotor = CANSparkMax(0, MotorType.kBrushless)
    private val rightMotor = CANSparkMax(1, MotorType.kBrushless)
    private val gyro = Pigeon2(0)

    fun driveForward(speed: Double) {
        leftMotor.set(speed)
        rightMotor.set(speed)
    }

    fun turn(degrees: Double) {
        // ooky spooky math
    }

    fun drive(forwards: Double, turn: Double) {
        // more ooky spooky math
    }
}
```

In this example, we have a `DriveSubsystem` that controls two motors and a gyro. It has methods to drive the robot
forward, turn, and drive in a specific direction. This is a pretty simple example, but you can make your subsystems as
complex as you need.

## Commands

Commands are what utilize those methods from the subsystems. You may be wondering why we don't just call the methods
from the subsystems directly, and the answer is that commands provide a "requirements" system that makes sure that
multiple commands aren't trying to control the same hardware at the same time.

Commands contain 4 methods, although you can skip some of them if you don't need them. The methods are:

- `initialize()`: This method is called when the command is first scheduled (not when its constructed). You can use it
  to set up anything you need for the command.
- `execute()`: This method is called repeatedly while the command is running. it runs every 20ms, so you can use it to
  control the hardware in your subsystems in a situation where the control request needs to change.
- `isFinished()`: This method is called repeatedly while the command is running. If it returns true, the command will
  end. If it returns false, the command will continue to run.
- `end()`: This method is called when the command ends. You can use it to clean up anything you need to.

You may have seen the word "scheduled" a few times in there. That's because commands are scheduled by
a `CommandScheduler`
object that runs in the background. This object keeps track of which subsystems are being used by which commands, and
makes sure that commands don't interfere with each other. It also keeps track of "Triggers" which are conditions that
can start, or schedule, commands. We'll discuss those in a minute.

Here's an example of a `DriveCommand` that drives a robot forward for a certain amount of time:

```kotlin
class DriveCommand(private val subsystem: DriveSubsystem, private val time: Double) : CommandBase() {
    init {
        // This tells the scheduler that this command requires the DriveSubsystem, so no other command can use it at the same time
        addRequirements(subsystem)
    }

    override fun initialize() {
        // Start driving forward
        subsystem.driveForward(0.5)
    }

    override fun isFinished(): Boolean {
        // Wait for the time to pass
        return timeSinceInitialized() >= time
    }

    override fun end() {
        // Stop driving
        subsystem.driveForward(0.0)
    }
}
```

As you can see commands can be pretty simple, or pretty complex. It all depends on what you need them to do.
Any form of robot control should be done using a Command

## Triggers

Triggers are essentially fancy booleans, they keep track of a condition and when that condition is met, they can start a
command. This is useful for things like starting a command when a button is pressed, or when a sensor reads a certain
value.

Commands can be bound to a command using several different conditions for scheduling, so here are a few examples:

- `.onTrue(command)`: This will start the command when the trigger is true.
- `.onFalse(command)`: This will start the command when the trigger becomes false.
- `.whileTrue(command)`: This will start the command when the trigger is true, and stop it when the trigger is false.
- `.whileFalse(command)`: This will start the command when the trigger is false, and stop it when the trigger is true.
- `.toggleOnTrue(command)`: This will start the command when the trigger is true, and stop it when the trigger is true
  again.
- `.toggleOnFalse(command)`: This will start the command when the trigger is false, and stop it when the trigger is
  false again, this operated when it CHANGES from false to true, so it wont be constantly flipping back and forth while
  the button or other condition is false

### Button Triggers

We talked about controllers in the last module, but there's extra classes we didn't talk about. the `CommandJoystick`
and `CommandXboxController` classes. These classes are essentially the same as the `Joystick` and `XboxController`
classes, however they return triggers instead of raw booleans, this makes it very easy to bind commands to buttons.

Here's an example of how you would bind a command to a button on an Xbox controller:

```kotlin
val controller = CommandXboxController(0)

controller.a().onTrue(DriveCommand(driveSubsystem, 2.0))
```

See! Super simple!

### Other Triggers

Triggers can be created manually via the `Trigger` class, and can be used to bind commands to any condition you can
think of. Here's an example of a trigger that starts a command when a sensor reads a value greater than 5:

```kotlin
val trigger = Trigger { sensor.get() > 5 }

trigger.onTrue(DriveCommand(driveSubsystem, 2.0))
```

Once again, super simple! Just make sure you use curly braces `{}` instead of parentheses `()` when creating the
trigger. This is because the `Trigger` class takes a lambda as a parameter, and lambdas are created using curly braces.
If you want to learn more about lambdas, you can check out
the [Kotlin documentation](https://kotlinlang.org/docs/lambdas.html).

## Command Composition

Alright, this is easily the most complex part of Command Based Programming, but it's also the most powerful. We don't
expect you to pick this up right away, but it's good to know that it exists. And start learning now.

Command Composition is the idea that you can create commands that are made up of other commands. This is useful for
creating complex sequences of commands that need to run in a specific order. For example, you might have
a `ShootCommand`
that is made up of a `SpinUpCommand`, a `FeedCommand`, and a `FireCommand`.

To create a command that is made up of other commands, you can use the `SequentialCommandGroup` class. This class takes
a list of commands as a parameter, and runs them one after the other. Here's an example of a `ShootCommand` that is made

up of a `SpinUpCommand`, a `FeedCommand`, and a `FireCommand`:

```kotlin
class ShootCommand : SequentialCommandGroup(
    init {
        addCommands(
            SpinUpCommand(shooterSubsystem),
            FeedCommand(feedSubsystem),
            FireCommand(fireSubsystem)
        )
    }
)
```

Or you can do it like this:

```kotlin
val ShootCommand = SequentialCommandGroup(
    SpinUpCommand(shooterSubsystem),
    FeedCommand(feedSubsystem),
    FireCommand(fireSubsystem)
)
```

There are other types of command groups, like `ParallelCommandGroup` which runs all of the commands at the same time,
`DeadlineCommand` which runs a command until a certain condition is met, and `RaceCommand` which runs all of the
commands
at the same time, but ends when the first command ends.

Theres one more way to compose commands, and it's the cleanest way to do it. It uses chained methods to create a command
group. Here's an example of how you would create a `ShootCommand` using this method:

```kotlin
val ShootCommand = SpinUpCommand(shooterSubsystem)
    .andThen(FeedCommand(feedSubsystem))
    .andThen(FireCommand(fireSubsystem))
```

The `andThen` method created a `SequentialCommandGroup`, `.alongWith` creates a `ParallelCommandGroup`, and so on and so
forth.

## Command Factories

For more complex commands with some form of internal state, you should use subclassing CommandBase. However, for simple
commands that don't need to store any state, you can use Command Factories. Command Factories are methods that return a
command, and are useful for creating super simple commands that don't need to store any state.

For example a way we could update the `DriveSubsystem` to have a `DriveCommand` that drives the robot forward for a
certain amount of time:

```kotlin
class DriveSubsystem : SubsystemBase() {
    private val leftMotor = CANSparkMax(0, MotorType.kBrushless)
    private val rightMotor = CANSparkMax(1, MotorType.kBrushless)

    fun driveForward(speed: Double) {
        leftMotor.set(speed)
        rightMotor.set(speed)
    }

    fun driveCommand(time: Double): Command {
        return this.run {
            driveForward(0.5)
        }.withTimeout(time)
    }
}
```

In this example, we added a `driveCommand` method to the `DriveSubsystem` that returns a command that drives the robot
forward for a certain amount of time. This is a super simple example, and that's the point. Command Factories are
useful for creating simple commands that don't need to store any state. If you need to store state, you should use
subclassing like we talked about earlier.

## Conclusion

Command Based Programming is a powerful way to structure your robot code. It makes it easy to write code that is
organized, easy to understand, and safe. It also provides a way to control complex systems like mechanisms and computer
vision pipelines.

We've only scratched the surface of Command Based Programming in this module. There's a lot more to learn, but this
should give you a good starting point. If you want to learn more (and you should), you can check out the WPILIB docs as
always. Some good starting points are the following

- [WPILIB Command Based Programming Docs](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html)