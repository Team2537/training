# General Code Practices

When writing code, following certain practices helps you and others understand it better. While basic formatting like newlines is handled by IntelliJ/Spotless (our code formatter), there are additional important practices to keep in mind when writing code on our team.

## Naming Conventions

- **Variables**: Use descriptive names to help understand what the code is doing.
    - Example: `distanceTraveled` instead of `x`.
    - Use camelCase: The first word is lowercase, and every subsequent word is capitalized.
- **Classes**: Use PascalCase: Every word is capitalized.
    - Example: `RobotBase`.

## Comments

- Use comments to explain why the code is doing something, not what it is doing. The code itself should be self-explanatory.

## Code Structure

- **Organization**: Group related code together. For example, place all code related to driving the robot in one section.

### Subsystem Structure

- **Properties and Methods**:
    - Place property-based data access (e.g., `get()` methods for small pieces of info) under normal properties but above methods.
    - Organize methods by functionality, grouping related methods together (e.g., all driving-related methods in one group).

## Documentation

- Document your code as you write it. This helps you and others understand your code and serves as a reference for future development.
- JavaDocs/Dokka:
    - It is encouraged to write JavaDocs/Dokka while working, but it is not mandatory for small changes.
    - For larger pull requests, JavaDocs/Dokka implementation is required.

## Code Review

- Code review is crucial for catching bugs and improving code quality.
- Have your code reviewed by at least one other person before merging it into the main branch.
- Software leads and mentors have the final say on what gets merged, but peer reviews are highly encouraged.

## Miscellaneous Code Practices

- Use property declaration in your constructor parameters.
- Prefer `val` over `var` whenever possible.
- Always use explicit types when declaring variables.
- Avoid using magic booleans; instead, use enums and `when` statements if possible.

# WPILIB Practices
_**TBD**_
