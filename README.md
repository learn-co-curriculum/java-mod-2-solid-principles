# SOLID Principles

## Learning Goals

- Explain the SOLID principles

## Introduction

"Just because you can doesn't mean you should"

This saying applies to many things in life, and it applies here in Object
Oriented Programming (and other types of programming) as well.

You could have deep hierarchies of classes that descend from a few base classes
in your system, and mistake inheritance as a mechanism to re-use code versus a
mechanism to reflect "is a" relationships in your system.

You could implement an entire system in a single class with many, many methods
calling each other and code being repeated in many places, not recognizing
simple patterns and logic that could simplify your code and your system.

There are many other mistakes you could make, some more subtle than others, and
Java gives us plenty of flexibility and power to do plenty of things that we
should not do.

There is a set of principles that were first popularized in the 90s and that
have hold up really well that we will introduce here to help you learn about
things you need to think about and consider when you're creating your classes,
your code and your system to solve whatever problems have been put in front of
you.

SOLID stands for:

1. S - Single Responsibility
2. O - Open/Closed Principle
3. L - Liskov Substitution
4. I - Interface Separation
5. D - Dependency Inversion

We will discuss each one of these principles below, but it's worth reflecting on
this reality of computer programming:

"Code is read many more times than it's written", which means we need to take
great care not just in making sure that our code works as desired, but also that
it's easy to understand, maintain and potentially change in the future.

## Single Responsibility

This principle dictates that a single class should only have one responsibility.
This does not mean a single class should only have one method, but rather that,
in general, it should only care about holding the properties relevant to its
definition and holding the logic relevant to the maintenance of those
properties.

Consider our sample `Student` class from earlier. We could have added the
numerical grade translation methods directly to that class, but it would have
started to a) making it larger and b) bleed its area of responsibility into
logic that is specific to the understanding of grades.

A good way to understand whether logic belongs in a specific class is to ask
whether there could be other entities in your system that would want to use the
functionality you're considering. In our student example, it's easy to imagine
that teachers and school administrators would also need to use the methods that
allow numerical grades to be translated into letter grades. A `Teacher` class
shouldn't have to inherit or have a dependency with a `Student` class in order
to be able to translate numerical grades into letter grades. Creating a class
that is specifically responsible for that logic allows us to avoid that
un-natural coupling for the `Student` class and for other classes in the future.

## Open/Closed Principle

"Open/Closed" refers to the principle that existing classes should be "open for
extension" but "closed for modification".

When new functionality must be introduced to an existing system and set of
classes, it is generally preferable to extend that system rather than modify its
existing behavior. This is because existing behavior may very well depend on
other parts of the system and have other parts of the system depend on it.

We will discuss unit testing in another section, and that has a significant
impact on our ability to modify existing code with confidence. However, this
principle really highlights that the more dependencies an existing piece of code
has, the wider the impact of changing it will be in your system. So even if you
are fully aware of the implications of the change you're making to existing
functionality, your changes will still have an impact on other dependent parts
of the system. Forcing other parts of the system to change just so you can
change your part of the system is increasing the work required to implement your
functionality.

This principle is aimed at reducing the impact of your work on other parts of
the system.

Let's consider our `Student` class, and imagine that we now want to be able to
manage information about students who are also athletes with the school. We
shouldn't modify the existing `Student` class to make it representative of
student athletes because a) not all students are athletes and b) student
athletes will have properties and methods that may drive them to respond
differently than students who aren't athletes. We should create a new class
`StudentAthlete` and have it either extend or make use of the `Student` class.

## Liskov Substitution

The Liskov Substitution Principle (LSP) dictates that "functions that use
references to base classes must be able to use objects of the derived class
without knowing it". Said more simply, we should be able to replace an instance
of a base class with an instance of one of its extension class in any method
that uses the base class and have the system continue to function the same way.

This is closely related to the Open/Closed principle in the previous section.
Because if a new subclass of a base class cannot be used by an existing method
without the existing method knowing it, then that existing method will need to
be modified so that it can handle both types differently. The changing of that
existing method violates the Open/Closed principle.

Let's consider the classic example of a `Rectangle` and a `Square` class. We
will start with our `Rectangle` implementation:

```java
public class Rectangle {
    private int width;
    private int height;

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

   public void printInfo() {
      System.out.println("I'm a rectangle (w: " + width + ", h: " + height + ")");
   }
}
```

This is a simple class with two `private` variables and `public` methods to get
and set those values (getters and setters)

We can use it in a simple program as follows:

```java
public class LSPDrawing {
    public static void main(String[] args) throws Exception {
        Rectangle rectangle = new Rectangle();
        rectangle.setHeight(10);
        rectangle.setWidth(20);
        rectangle.printInfo();
        transform(rectangle, 5, 10);
        draw(rectangle);
    }

    static void transform(Rectangle rectangle, int newHeight, int newWidth) throws Exception {
        rectangle.setHeight(newHeight);
        rectangle.setWidth(newWidth);

        if (rectangle.getWidth() != newWidth || rectangle.getHeight() != newHeight) {
            throw new Exception("This rectangle has been corrupted");
        }
    }

    static void draw(Rectangle rectangle) {
        System.out.println("drawing (w: " + rectangle.getWidth() + ", h: " + rectangle.getHeight() + ")");
    }
}
```

The `LSPDrawing` class simply creates a rectangle, sets its dimensions, prints
its information, applies a transformation and draws it. The `transform` method
does something interesting that we will get back to a little bit later.

Now we need to model a square. When we think about inheritance and the "is a"
relationships we discussed earlier, we immediately think that "a square is a
rectangle", right? A square is just a special version of a rectangle where the
sides are always equal. So we might start like this:

```java
public class Square extends Rectangle {
   @Override
   public void printInfo() {
      System.out.println("I'm a square (w/l: " + getWidth() + ")");
   }
}
```

But since a square must have equal length sides in order to be a square, our
current implementation has a problem. If someone called the `setWidth()` method
on an instance of the `Square` class, they would end up with a `Square` object
that does not necessarily have the same `width` as it does `height`.

We can fix this issue by overriding the `setWidth()` and `setHeight()` methods
on our `Square` class to ensure they enforce this new rule though, which will
give us a `Square` class like this:

```java
public class Square extends Rectangle {

    @Override
    public void setHeight(int height) {
        super.setHeight(height);
        super.setWidth(height);
    }

    @Override
    public void setWidth(int width) {
        super.setHeight(width);
        super.setWidth(width);
    }

    @Override
    public void printInfo() {
        System.out.println("I'm a square (w/l: " + getWidth() + ")");
    }
}
```

Now every time we set the `width`, we make sure we also set the `height` to the
same value, thus enforcing the rules of a square object.

It appears that all is good and that we should be able to use the `Square` class
just like we had been using the `Rectangle` class, but we cannot...

Let's get back to the `transform` method we mentioned earlier:

```java
    static void transform(Rectangle rectangle, int newHeight, int newWidth) throws Exception {
        rectangle.setHeight(newHeight);
        rectangle.setWidth(newWidth);

        if (rectangle.getWidth() != newWidth || rectangle.getHeight() != newHeight) {
            throw new Exception("This rectangle has been corrupted");
        }
    }
```

In this method, the programmer wanted to be sure that `setWidth()` and
`setHeight()` were doing their job properly and that transforming the
`rectangle` it receives in the parameters had the desired effect, so they
checked that the `width` is indeed the `newWidth` and that the height is indeed
the `newHeight` after the appropriate calls are made.

Note: this is somewhat of a simplified example, and getters and setters would
not normally be tested in this way. But the underlying concept that a class that
uses another class would engage in defensive programming to ensure that its
actions had the desired effects based on assumptions it's making about the
objects in play is reasonable and does happen all the time in real systems.

Here is our runner class with an added `Square` object in it:

```java
public class LSPDrawing {
    public static void main(String[] args) throws Exception {
        System.out.println("Running...");

        Rectangle rectangle = new Rectangle();
        rectangle.setHeight(10);
        rectangle.setWidth(20);
        rectangle.printInfo();
        transform(rectangle, 5, 10);
        draw(rectangle);

        Square square = new Square();
        square.setHeight(5);
        square.printInfo();
        transform(square, 10, 20);
        square.printInfo();
        draw(square);
    }

    static void transform(Rectangle rectangle, int newHeight, int newWidth) throws Exception {
        rectangle.setHeight(newHeight);
        rectangle.setWidth(newWidth);

        if (rectangle.getWidth() != newWidth || rectangle.getHeight() != newHeight) {
            throw new Exception("This rectangle has been corrupted");
        }
    }

    static void stretch(Rectangle rectangle, int stretchFactor) {
        rectangle.setWidth(rectangle.getWidth() * stretchFactor);
        rectangle.setHeight(rectangle.getHeight() * stretchFactor);
    }

    static void draw(Rectangle rectangle) {
        System.out.println("drawing (w: " + rectangle.getWidth() + ", h: " + rectangle.getHeight() + ")");
    }
}
```

This code will not run to the end because the `transform()` method will throw an
exception, since the `setWidth()` method of our `Square` class actually sets
both the `width` and the `height` and therefore breaks the assumption that was
made when the `transform()` method was written.

This problem can be fixed by changing the `transform()` method to perform its
check differently based on whether the incoming object is actually a `Square` or
a `Rectangle`, but the point is that it _will_ have to be change in order to run
properly.

This new `Square` class violates the Liskov Substitution principle because it
cannot be used everywhere where its superclass, the `Rectangle` class, can be
used without changes.

What this reveals is that a `Square` isn't really a `Rectangle`. Yes, a square
has the same properties as a rectangle in that it has 4 sides and they all
intersect at a 90-degree angle. But a square does not behave like a rectangle.
If you grab one of the sides of a rectangle in your favorite drawing program,
you will be able to stretch that side without impacting the other dimension
(width vs height). Do the same thing to a square, and all sides will stretch at
the same time, or they should if the square is to stay a square.

When considering the "is a" relationship between 2 classes, make sure to
consider their respective behavior in addition to their properties.

Another important learning from examining this example is that considering a
class by itself, without considering how it's being used, is not a reliable way
to determine its validity. Once we extended the `Rectangle` class to create the
`Square` class, we could see issues with the way that `width` and `height` are
handled right away, but once we addressed those specific issues, looking at each
class in isolation would have led us to believe that both classes had now been
fixed and that the system as a whole would work properly.

## Interface Separation

As you consider the existing requirements of a system to design a solution, it
is important to remember that we, humans, are very bad at predicting the future.
Given that we're not good at predicting the future, we should be careful not to
design our systems in such a way that they either a) make significant
assumptions about the future or b) make it very difficult to react to future
conditions that we did not anticipate.

One way to keep flexibility in our system and make sure we don't let our current
understanding of the future paint us in a corner is to make sure our class
hierarchy is as specific to what we know as possible.

Consider our student athlete example from earlier. We could model a student
athlete with the following interface:

```java
public interface StudentAthlete {
    void study();
    void play();
    void walk();
}
```

These are all things that a `StudentAthlete` can reasonably be expected to do,
but now my `play()` functionality is tied to the same interface as my `study()`
functionality. What if I have a student who isn't an athlete or an athlete who
is not a student?

Instead of designing my interfaces around what their implementation might be,
like a `StudentAthlete`, I should design my interfaces around the capabilities I
want their implementations to have, and I should make sure those capabilities
are separated so they can be combined however they need do by implementing
classes.

In this example, I might have a `SportsPlayer` interface that holds the `play()`
method, a `HumanMover` interface that holds the `walk()` method and a `Student`
interface that holds the `study()` method.

Then my `StudentAthlete` can now implement all 3 interfaces:

```java
public class StudentAthlete implements SportsPlayer, HumanMover, Student {
   void play() { ... }
   void walk() { ... }
   void study() { .... }
}
```

This also allows me to have a `ProfessionalAthlete` class that only implements 2
of the same interfaces:

```java
public class ProfessionalAthlete implements SportsPlayer, HumanMover {
   void play() { ... }
   void walk() { ... }
}
```

Furthermore, we can also think of additional capability that a professional
athlete has that student athletes do not, such as the ability to endorse
products, for example. When we abstract that ability with another specific
interface:

```java
public interface ProductEndorser {
   void signContract();
   void receivePayment();
   void announceEndorsement();
}
```

So now we have a `ProfessionalAthlete` class that looks like this:

```java
public class ProfessionalAthlete implements SportsPlayer, HumanMover, ProductEndorser {
   void play() { ... }
   void walk() { ... }
   void signContract() { ... }
   void receivePayment() { ... }
   void announceEndorsement() { ... }
}
```

Finally, consider that professional athletes aren't the only people through whom
companies are interested to advertise their products. Imagine that we are now
asked to model that artists can also be product endorsers. Since our
`ProductEndorser` interface stands on its own and does not carry with it any
functionality not related to product endorsement, we can simply make our
`Artist` class implement our `ProductEndorser` interface (in addition to
whatever other interfaces it might already implement):

```java
public class Artist implements ProductEndorser, ... {
        void signContract() { ... }
        void receivePayment() { ... }
        void announceEndorsement() { ... }

        ...
}
```

## Dependency Inversion

The dependency inversion principle states that high level modules should not
depend on low level modules, that they should both depend on abstractions, and
that abstractions should not depend on details but rather details should depend
on abstractions.

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

We can now have a project runner that looks like this:

```java
public class TechnicalProjectRunner {
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
   know how to code in Python and Android, respectively
2. Create a different version of `TechnicalProject` that uses those new
   implementations of backend and frontend developer
3. Finally change `TechnicalProjectRunner` so that it uses the different version
   of `TechnicalProject`

But in reality, the only change I should need to make is ensure I have Python
and Android developers and that I can specify that I want to use them instead of
the Java and Javascript developers I'm currently using. I should be able to keep
everything else the same.

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
   might come up with in the future
2. Instead of creating our own developers, we're going to let the caller of the
   `startDelivery()` method tell us which implementation they want us to use for
   that particular delivery

```java
public class TechnicalProject {
    public void startDelivery(Developer backendDev, Developer frontendDev) {
        backendDev.writeCode();
        frontendDev.writeCode();
    }
}
```

We can now change the `TechnicalProjectRunner` class to create the kind of
developer it wants to use and request to start the delivery of the project:

```java
public class TechnicalProjectRunner {
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

```
Writing code in Java
Writing code in Javascript
```

Now that we have this new baseline, let's consider our extended requirements
again - we want to be able to use Python developers for the backend and Android
developers for the frontend.

Instead of having to make all the changes we needed with the initial design, we
can now simply do the following:

1. Create different implementations of `Developer` to support Python and Android
2. Change `TechnicalProjectRunner` to take advantage of those new classes

We eliminated an entire step from before, and also concentrated the changes in
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
public class TechnicalProjectRunner {
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

Following the dependency inversion principle ensures that you are building a
flexible system that does not make too many assumptions about the future and can
easily adapt when requirements change or are extended.
