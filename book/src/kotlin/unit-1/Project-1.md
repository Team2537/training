# Unit 1 Project: Movie Collection

In this project, you will create a simple movie collection program. The program will allow you to add movies to a
collection, view the movies in the collection, and search for movies by title.

### Project Requirements

1. Create a `Movie` class with the following properties:
    - `title`: The title of the movie
    - `year`: The year the movie was released
    - `genre`: The genre of the movie
    - `rating`: The rating of the movie
      The `Movie` class should have a primary constructor that initializes these properties.
    - The properties should be read-only (use `val` instead of `var`).
    - The `rating` property should be a `Double` value between `0.0` and `10.0`.
    - The `year` property should be an `Int` value representing the release year.
    - The `genre` property should be a `String` value representing the genre of the movie.
    - The `title` property should be a `String` value representing the title of the movie.
    - A `display` method that returns a string representation of the movie in the
      format: `title (year) - genre - rating`.

2. Create a `MovieCollection` class to manage a collection of movies. The `MovieCollection` class should allow you to:
    - Add a movie to the collection.
    - View all movies in the collection.
    - Search for a movie by title.
    - Remove a movie from the collection by title.
    - Calculate the average rating of all movies in the collection.
    - Calculate the number of movies in the collection.
    - Clear the collection (remove all movies).

3. Create a `main` function to test your `MovieCollection` class. In the `main` function, create an instance of
   the `MovieCollection` class and test the functionality of adding movies, viewing movies, searching for movies,
   removing movies, calculating the average rating, calculating the number of movies, and clearing the collection.

Obviously this isn't strictly "graded" but you should be able to demonstrate the functionality of your program to your
software leads, as well as an understanding of the basic concepts of Kotlin.

It is recommended to use GitHub for this project, as you will need to submit it using github.

Once you have completed the project, send a message on Slack to your software leads with a link to the repository
containing your project, so we can review your code.
