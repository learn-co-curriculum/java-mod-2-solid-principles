# SOLID Principles

## Learning Goals

- Explain the SOLID principles

## Introduction

"Just because you can doesn't mean you should"

This saying applies to many things in life, and it applies here in
Object-Oriented Programming (and other types of programming) as well.

We could have deep hierarchies of classes that descend from a few base classes
in our system and completely abuse the pillar of inheritance.

We could implement an entire system in a single class with many, many methods
calling each other and code being repeated in several places.

There are many other mistakes we could make, some more subtle than others, and
Java gives us plenty of flexibility and power to do plenty of things that we
should not do.

There is a set of principles that were first popularized in the 90s and that
we can and should follow when programming. These principles are known as the
**SOLID principles**.

SOLID stands for:

1. S - Single Responsibility
2. O - Open/Closed Principle
3. L - Liskov Substitution
4. I - Interface Separation
5. D - Dependency Inversion

We will discuss each one of these principles below, but it's worth reflecting on
this reality of computer programming:

"Code is read many more times than it's written", which means we need to take
great care of not just in making sure that our code works as desired, but also
that it's easy to understand, maintain and potentially change in the future.

## Single Responsibility

This principle dictates that a single class should only have one responsibility.
This does not mean a single class should only have one method, but rather that,
in general, it should only care about holding the properties relevant to its
definition and holding the logic relevant to the maintenance of those
properties.

We can directly relate this back to the Coupling and Cohesion lesson when we
were talking about the difference between high and low cohesion with a `User`
class. The `User` class should only be responsible for methods and attributes
that are directly related and associated with the `User` object. A good way to
understand whether logic belongs in a specific class is to ask whether there
could be other entities in a system that would want to use the functionality.

## Open/Closed Principle

"Open/Closed" refers to the principle that existing classes should be "open for
extension" but "closed for modification".

When new functionality must be introduced to an existing system, it is
generally preferable to extend that system rather than modify its
existing behavior. This is because existing behavior may very well have
dependencies linked to it.

Consider a `Book` class and assume it has already been implemented in the
application for a while:

```java
public class Book {
    String author;
    String title;
    String genre;
    
    // Constructors, getters, setters, and other methods
}
```

Maybe after some time has passed, we decide we want a textbook component that
would introduce the attributes of a volume and an edition. Instead of modifying
the `Book` class, and possibly breaking some code that is dependent on it, we
will extend it as per this principle:

```java
public class Textbook extends Book {
    String volume;
    String edition;
    
    // Constructors, getters, setters, and other methods
}
```

Extending the `Book` class into the `Textbook` class ensures the existing
application will not be affected.

## Liskov Substitution

The Liskov Substitution Principle (LSP) dictates that "functions that use
references to base classes must be able to use objects of the derived class
without knowing it". Said more simply, we should be able to replace an instance
of a parent class with an instance of one of its child classes in any method
that uses the parent class and have the system continue to function the same way.

This is closely related to the Open/Closed principle. If a new subclass of a
parent class cannot be used by an existing method without the existing method
knowing it, then that existing method will need to be modified so that it can
handle both types differently. The changing of that existing method violates the
Open/Closed principle.

Consider we have a `Car` interface, and we have a `GasCar` class that
implements the `Car` interface:

```java
public interface Car {
    void turnOnEngine();
    void turnOffEngine();
}
```

```java
public class GasCar implements Car {
    private Engine engine;
    
    @Override
   public void turnOnEngine() {
        engine.start();
    }
    
    @Override
    public void turnOffEngine() {
        engine.stop();
    }
}
```

Nowadays, we have electric cars that don't take gas... and don't have an engine.
Let's see what happens when we create an `ElectricCar` class and have it
implement the `Car` interface:

```java
public class ElectricCar implements Car {
    
    @Override
   public void turnOnEngine() {
        // Oh no! I don't have an engine to turn on!
    }
    
    @Override
    public void turnOffEngine() {
        // Oh no! I don't have an engine to turn off!
    }
}
```

This is a violation of our Liskov Substitution principle and will be harder to
integrate into the existing model based on the Open/Closed principle. When
considering the "is a" relationship between 2 classes, make sure to consider
their respective behavior in addition to their properties.

## Interface Separation

As we consider the existing requirements of a system to design a solution, it
is important to remember that we, humans, are very bad at predicting the future.
Given that we're not good at predicting the future, we should be careful not to
design our systems in such a way that they either make significant assumptions
about the future or make it very difficult to react to future conditions that we
did not anticipate.

One way to keep flexibility in our system and make sure we don't let our current
understanding of the future paint us in a corner is to make sure our class
hierarchy is as specific to what we know as possible.

Consider a `StudentAthlete` interface. We could model a student athlete with the
following interface:

```java
public interface StudentAthlete {
    void study();
    void play();
    void walk();
}
```

These are all things that a `StudentAthlete` can reasonably be expected to do,
but now the `play()` functionality is tied to the same interface as the
`study()` functionality. What if we have a student who isn't an athlete or an
athlete who is not a student?

Instead of designing the interfaces around what their implementation might be,
like a `StudentAthlete`, we should design our interfaces around the capabilities
we want their implementations to have. We should also make sure those
capabilities are separated, so they can be combined however they need do by
implementing classes.

In this example, we might have a `SportsPlayer` interface that holds the `play()`
method, a `HumanMover` interface that holds the `walk()` method, and a `Student`
interface that holds the `study()` method.

Then our `StudentAthlete` can now implement all 3 interfaces:

```java
public class StudentAthlete implements SportsPlayer, HumanMover, Student {
   void play() { ... }
   void walk() { ... }
   void study() { .... }
}
```

This also allows us to have a `ProfessionalAthlete` class that only implements 2
of the same interfaces:

```java
public class ProfessionalAthlete implements SportsPlayer, HumanMover {
   void play() { ... }
   void walk() { ... }
}
```

## Dependency Inversion

The dependency inversion principle refers to decoupling software modules.
So, high-level modules won't depend on low-level modules. Instead, both will
depend on abstractions.

Let's consider the example of technical project that requires both some work on
the backend (data processing, API work, ...) and work on the frontend (creating
a user interface and handling user interaction). For this example, we'll assume
the backend work is done using Java and the frontend work is done using
Javascript.

We could model a `BackendDeveloper` class like this:

```java
public class BackendDeveloper {
    void writeCode() {
        System.out.println("Writing code in Java");
    }
}
```

And model a `FrontendDeveloper` class like this:

```java
public class FrontendDeveloper {
    void writeCode() {
        System.out.println("Writing code in Javascript");
    }
}
```

We could then use both of those classes in a `TechnicalProject` class like this:

```java
package com.flatiron.dip;

public class TechnicalProject {
    private BackendDeveloper backendDev = new BackendDeveloper();
    private FrontendDeveloper frontendDev = new FrontendDeveloper();

    public void startDelivery() {
        backendDev.writeCode();
        frontendDev.writeCode();
    }
}
```

This gives us a functional representation of a technical project on which we can
start delivery by asking both our developers to start coding.

We can now have a driver class that looks like this:

```java
public class TechnicalProjectDriver {
    public static void main(String[] args) {
        TechnicalProject myProject = new TechnicalProject();
        myProject.startDelivery();
    }
}
```

Running this code gives us the following output:

```plaintext
Writing code in Java
Writing code in Javascript
```

The problem is that the technical project now has a tightly coupled dependency
on the specific implementations of both the backend and the frontend developer.
What if we wanted to run a technical project with Python developers for the
backend and Android developers for the frontend?

As things stand, we would need the following:

1. Create different classes for `BackendDeveloper` and `FrontendDeveloper` that
   know how to code in Python and Android, respectively.
2. Create a different version of the `TechnicalProject` that uses those new
   implementations of backend and frontend developers.
3. Finally, change `TechnicalProjectDriver` so that it uses the different version
   of `TechnicalProject`.

But in reality, the only change we should need to make is ensure we have Python
and Android developers and that we can specify that we want to use them instead
of the Java and Javascript developers. We should be able to keep everything else
the same.

This can be done using dependency inversion.

First, let's create an interface that defines what a developer does, which we'll
over simplify by saying a developer "writes code":

```java
public interface Developer {
    void writeCode();
}
```

We will now make our existing `BackendDeveloper` and `FrontendDeveloper`
implement this interface:

```java
public class BackendDeveloper implements Developer {
    public void writeCode() {
        System.out.println("Writing code in Java");
    }
}
```

```java
public class FrontendDeveloper implements Developer {
    public void writeCode() {
        System.out.println("Writing code in Javascript");
    }
}
```

We can then change our `TechnicalProject` in two ways:

1. Instead of using the implementations of the `Developer` interface, it will
   use the interfaces, making it flexible to use whatever other implications we
   might come up with in the future.
2. Instead of creating our own developers, we're going to let the caller of the
   `startDelivery()` method tell us which implementation they want us to use for
   that particular delivery.

```java
public class TechnicalProject {
    public void startDelivery(Developer backendDev, Developer frontendDev) {
        backendDev.writeCode();
        frontendDev.writeCode();
    }
}
```

We can now change the `TechnicalProjectDriver` class to create the kind of
developer it wants and request to start the delivery of the project:

```java
public class TechnicalProjectDriver {
    public static void main(String[] args) {
        TechnicalProject myProject = new TechnicalProject();
        Developer backendDev = new BackendDeveloper();
        Developer frontendDev = new FrontendDeveloper();

        myProject.startDelivery(backendDev, frontendDev);
    }
}
```

At this point, we have created a different version of the code we had before
that was able to start delivery of a project by writing Java and Javascript
code, so running this new version yields the exact same output as before:

```plaintext
Writing code in Java
Writing code in Javascript
```

Now that we have this new baseline, let's consider our extended requirements
again - we want to be able to use Python developers for the backend and Android
developers for the frontend.

Instead of having to make all the changes we needed with the initial design, we
can now simply do the following:

1. Create different implementations of `Developer` to support Python and Android
2. Change `TechnicalProjectDriver` to take advantage of those new classes

We eliminated an entire step from before and also concentrated the changes in
areas where they are actually impactful.

We now have 2 new classes because we need to support 2 new types of developers:

```java
public class PythonDeveloper implements Developer {
    public void writeCode() {
        System.out.println("Writing code in Python");
    }
}
```

```java
public class AndroidDeveloper implements Developer {
    public void writeCode() {
        System.out.println("Writing code for Android");
    }
}
```

And we can make very minimal changes to our project running to take advantage of
the new types of developers:

```java
public class TechnicalProjectDriver {
    public static void main(String[] args) {
        TechnicalProject myProject = new TechnicalProject();
        Developer backendDev = new PythonDeveloper(); // <---- use PythonDeveloper instead of BackendDeveloper
        Developer frontendDev = new AndroidDeveloper(); // <---- use AndroidDeveloper instead of FrontendDeveloper

        myProject.startDelivery(backendDev, frontendDev);
    }
}
```

With these changes, we now get the following output:

```plaintext
Writing code in Python
Writing code for Android
```

Following the dependency inversion principle ensures that we are building a
flexible system that does not make too many assumptions about the future and can
easily adapt when requirements change or are extended.
