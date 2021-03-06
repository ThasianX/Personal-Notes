# [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)

### Problem

- Fitting together web controller architecture with database interface backing when they were built by different teams with little knowledge of each other

### Components and services

- Definitions
    - Component
        - Software that's intended to be used, without change, by an application that is out of the control of the writers of the component
        - Without change meaning that the application doesn't change the source code of the components, though they may extend upon the behaviors of the component
    - Service
        - Similar to a component in that it's used by foreign applications
        - Used remotely through some remote interface, either synchronous or asynchronous

### Naive example

    public interface MovieFinder {
    List findAll();
    }

    class MovieLister...

    private MovieFinder finder;
    public MovieLister() {
    finder = new ColonDelimitedMovieFinder("movies1.txt");
    }

    public Movie[] moviesDirectedBy(String arg) {
    List allMovies = finder.findAll();
    for (Iterator it = allMovies.iterator(); it.hasNext();) {
    Movie movie = (Movie) it.next();
    if (!movie.getDirector().equals(arg)) it.remove();
    }
    return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);
    }

- The interface for the finder is well decoupled
- moviesDirectedBy is tightly coupled with how movies are being stored, in this case through the finder
    - This becomes a problem if other people also want to use your movie finder but instead of storing their movie listings in a colon delimited file called "movies1.txt", they use something else like SQL or XML
    - Then, we need a different class to grab that data

    <img src="./movielister.png" width="400" height="300">

- Based on the above figure, the question is how do you make it so that the lister class is ignorant of the implementation class but can still talk to an instance to do its work
    - In real systems, you can abstract the use of components by talking to them through an interface.
    - But deploying the system in different ways would require plugins to handle the interaction with these services so they can use different implementations
    - These plugins for an application are assembled through inversion of control

### Inversion of control

- Common characteristic of frameworks
- What aspect of control are you inverting?
    - For this new breed of containers, the inversion is how they lookup a plugin implementation
        - In the naive example, the lister looked up the finder implementation by directly instantiating it which stops it from being a plugin
- The approach
    - Ensure that any user of a plugin follows some convention that allows a separate assembler module to inject the implementation into the lister
- New name
    - IoC is too generic
    - The general consensus now is *Dependency Injection*

### Forms of dependency injection

- Basic idea
    - Have a separate object that populates a field in the lister class with an appropriate implementation for the finder interface

        <img src="./injectiondependencies.png" width="500" height="300">

- Constructor injection with PicoContainer
    - PicoContainer
        - Uses constructor to decide how to inject a finder implementation into the lister class
    - Process
        - Create a constructor in movie lister that includes it needs injected
        - Create a parameter for the filename of the text file for the finder's constructor
        - Pico container then determines which implementation to associate with each interface and which string to inject into the finder
- Setter injection with Spring
    - Process
        - Define a setting method for the finder in MovieLister
        - Define a setter in the file name for the MovieFinder
        - Set up the configuration for the files
            - Spring supports configuration through XML and through code
- Interface injection
    - Process
        - Define an interface to perform the injection through
        - This interface would be defined by the class that provides the MovieFinder instance
        - Have MovieLister, the class that wants to use a finder, implement it

### Using a service locator

- Basic idea
    - Have an object that knows how to get hold of all the services that an application might need
    - For this application, there would be a method that returns a movie finder when one is needed

    <img src="./servicelocatordependencies.png" width="400" height="300">
- Service locator as a Registry
    - MovieLister calls on a method in the ServiceLocator when it's instantiated
    - Problem
        - MovieLister is dependent on the full service locator class, even though it only uses 1 service
- Using a segregated interface for the locator
    - Addresses the service locator problem above through a role interface
    - The MovieLister can declare just the bit of interface it needs from the locator interface
    - Process
        - The provider of the MovieLister would also need to provide a locator interface which it needs to get hold of the finder
        - The locator then needs to implement this interface to provide access to a finder
- A dynamic service locator
    - Basic idea
        - Allows you to stash any service you need into it and make your choices at runtime
        - Uses a map instead of fields for each of the services and provides generic methods to get and load services
    - Drawbacks
        - Not as explicit as a segregated interface
- Using both a locator and injection
    - Example
        - Avalon uses injection to tell components where to find the service locator
        - Use interface injection to inject a service manager into a class

## Deciding which option to use

### Service locator vs dependency injection

- Both provide the fundamental decoupling missing from the naive approach
- Difference
    - Service locator
        - The application class of the service locator asks for the implementation explicitly by a message to the locator
        - Every user of a service has a dependency to the locator
    - Dependency injection
        - In injection, the service appears in the application class, hence the inversion of control
        - Hard to understand at times and leads to problems while debugging
        - Makes it easier to see what the component dependencies are
- When to use which
    - Service locator
        - Applications with various classes that use a service
    - Dependency injection
        - Providing to an application that other people are writing to
        - Component cannot obtain further services from the injector once it's been configured

### Constructor vs setter injection

- Constructor injection
    - Constructors with multiple parameters give you a clear statement of what it means to create a valid object
    - Allows you to clearly hide any fields that are immutable by simply not providing a setter
    - Cons
        - Can get a bit out of hand when there are multiple ways to construct an objects
- Factory methods
    - Use a combination of private constructors and setters to implement their work
