# Dokka / KDocs

Kotlin provides a very nice tool called `Dokka` that is used to generate a website that documents your code. This site
is generated from specially formatted comments in your code, called `KDocs`. These comments are written in a special
format that allows Dokka to understand them and generate a website from them.

Here's an example of a KDoc comment on a function:

```kotlin
/**
 * This is a KDoc comment. It is used to document your code.
 * 
 * @param x The x coordinate of the point.
 * @param y The y coordinate of the point.
 * @return The distance from the origin to the point.
 */
fun distanceFromOrigin(x: Double, y: Double): Double {
    return sqrt(x * x + y * y)
}
```

As you can see its a normal multi-line comment, but it starts with `/**` instead of `/*`. This is how Kotlin knows that
this is a KDoc comment. Inside the comment, you can use special tags like `@param` to document the parameters of the
function, and `@return` to document the return value of the function. These tags are used by Dokka to generate the
documentation website.

You can also document classes, properties, and other things in Kotlin using KDocs. Here's an example of a KDoc comment on
a class:

```kotlin
/**
 * This is a KDoc comment. It is used to document your code.
 * 
 * @property x The x coordinate of the point.
 * @property y The y coordinate of the point.
 * @constructor Creates a new Point with the given coordinates.
 */
class Point(val x: Double, val y: Double) {
    // Class implementation goes here
}
```

As you can see, you can use the `@property` tag to document the properties of the class, and the `@constructor` tag to
document the constructor of the class. These tags are used by Dokka to generate the documentation website.

The rest of the tags you can use in KDocs, as well as more information on how to write KDocs, can be found in the
[KDocs documentation](https://kotlinlang.org/docs/kotlin-doc.html).