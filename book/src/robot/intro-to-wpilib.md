# Intro to WPILIB and its Features

WPILIB is a library that provides a set of tools and utilities for programming robots in Java, C++, and Kotlin. It is
the primary library used for programming FRC robots and provides a wide range of features to help you develop your robot
code.

In this chapter, we will cover the basics of WPILIB and its utility features, including:

- Typesafe Units
- Geometry Classes
- Math Utilities

All of these features are designed to make programming your robot easier and more efficient. Let's dive in!

## Typesafe Units

One of the problems that can arise when programming things tha operate in the real world is that units can get mixed up.
For example, if you are working with distances, you might accidentally mix up inches and meters. This can lead to bugs
in
your code that are difficult to track down.

WPILIB provides a set of typesafe units that help you avoid these issues. By "typesafe", we mean that different types of
units like distances, angles, and velocities are represented by different classes. This makes it impossible to mix up
units in your code.

The basics of using them is as follows:

```kotlin
val distance: Measure<Distance> = Units.Feet.of(10.0) // 10 feet

val angle: Measure<Angle> = Units.Degrees.of(90.0) // 90 degrees

val velocity: Measure<Velocity<Distance>> = Units.FeetPerSecond.of(5.0) // 5 feet per second
```

As you can see all units are a form of `Measure` with a specified type. This makes it easy to work with different units.
Then, you can use the `of` method to create a new instance from a value.

There's a lot of other interesting things you can do with units, but this is the basic idea.
To learn more about units, you can check out
the [WPILIB documentation](https://docs.wpilib.org/en/latest/docs/software/basic-programming/java-units.html).

## Geometry Classes

WPILIB provides a set of classes that allow you to represent things like points, rotations, and poses in 2D and 3D
space.
These classes are useful for working with robot geometry and kinematics, and tie in nicely with the typesafe units we
just talked about.

The table below shows some of the classes that WPILIB provides:

| Class           | Description                                               |
|-----------------|-----------------------------------------------------------|
| `Translation2d` | Represents a 2D point with x and y coordinates.           |
| `Rotation2d`    | Represents a 2D rotation using a point on the unit circle |
| `Pose2d`        | Represents a 2D pose with a translation and rotation.     |
| `Translation3d` | Represents a 3D point with x, y, and z coordinates.       |
| `Rotation3d`    | Represents a 3D rotation using a quaternion.              |
| `Pose3d`        | Represents a 3D pose with a translation and rotation.     |

These classes are useful for representing things like robot positions and orientations, and can be used in conjunction
with the typesafe units we talked about earlier.

You can learn more about these classes in
the [WPILIB documentation](https://docs.wpilib.org/en/latest/docs/software/advanced-controls/geometry/index.html).

## Math Utilities

At the moment WPILIB only really provides one math utility, and its made obsolete in a lot of cases because of the
Typesafe Units. It provides a way to convert between different units stored as doubles, but you should use the typesafe
units whenever possible.

Most of these methods are formatted as `unit1ToUnit2` and are pretty self-explanatory. For example, to convert feet to
meters, you would use `feetToMeters` and pass in a double.

Although there isn't a real page for this in the WPILIB documentation, you can find the methods in
the [WPILIB JavaDocs](https://github.wpilib.org/allwpilib/docs/release/java/edu/wpi/first/math/util/Units.html).