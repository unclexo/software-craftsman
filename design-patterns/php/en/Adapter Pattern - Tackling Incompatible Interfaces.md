![Adapter Pattern - Tackling Incompatible Interfaces](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-cover-photo.jpg)

# Adapter Pattern - Tackling Incompatible Interfaces
Today we are going to talk about the Adapter Design Pattern. This is one of the most used software design patterns. 
We will explore when to use this pattern and how it helps us tackle classes and objects that have incompatible 
interfaces. We will start drawing this article with the problem that may be solved by the Adapter Pattern.

## The Problem
Let's say we have an existing web application where a client code expects a vehicle object. If we provide a vehicle 
object to the client code then the code may start the engine and set the speed of that vehicle. Let's see such 
a client code.

```php
<?php

function start(VehicleInterface $vehicle)
{
    $vehicle->startEngine();

    $vehicle->speed();
}
```
In our example, the client code is a simple function. But keep in mind, it could be a controller method or any class 
method of a module.

Let's try to start a vehicle. Say we have a different kind of vehicles class written in our existing web application. 
The `MotorBike` class is one of them, for example. Let's see the implementation details of the `MotorBike` class:

```php
<?php

class MotorBike implements VehicleInterface 
{
    public function startEngine() 
    {
        var_dump('The Engine has been started...');   
    }
    public function speed() 
    {
        var_dump("Setting the speed to 100 km per hour.");
    }
}
```

And the `VehicleInterface` interface that has been implemented by the `MotorBike` class would look like the below:

```php
<?php

interface VehicleInterface 
{
    public function startEngine();

    public function speed();
}
```

Now we will try to start the `MotorBike` via the client code. Check out below:

```php
<?php

$motorBike = new MotorBike();

start($motorBike);
```

If we run the script above, we get the following outputs:

```php
<?php

// Outputs

The Engine has been started...

Setting the speed to 100 km per hour.
```

Therefore, the Motor Bike has been started and running at 100 km per hour.

Everything is Okay.

But suddenly we heard from the project manager of the web application that we need to use a new vendor class
`RoadBike` in the client code so that, from now on, the client code supports bicycle object too. Moreover, the new 
vendor class implements a different interface. Let's see the `RoadBike` class below:

```php
<?php

class RoadBike implements BicycleInterface 
{
    public function pedal() 
    {
        var_dump('The pedaling has been started...');    
    }
    public function speed() 
    { 
        var_dump("Setting the speed to 20 km per hour."); 
    }
}
```

And the `BicycleInterface` interface would look like the one below:

```php
<?php

interface BicycleInterface 
{
    public function pedal();

    public function speed();
}
```

We see. The `BicycleInterface` is different from `VehicleInterface` and it has a different method named `pedal()`. 
Otherwise, a `RoadBike` is a bicycle so it has no engine like a `MotorBike`.

What do we do?

The project manager has also said we can NOT modify our existing client code. Because that we will break the 
[Open-Closed Principle](https://medium.com/@unclexo/ocp-the-open-closed-principle-33eab31c7b92). As the `RoadBike` 
class has a different interface so we can NOT use this directly within the client code above. So this is a problem.

What is the solution?

The solution is that we need a middleman - a class that can help us solve the problem. A class that can adapt the 
`RoadBike` class or its interface (or what the `RoadBike` class can do) into the existing client code. A class that can 
convert the interface of the `RoadBike` class so that the `RoadBike` class can be used in the client code in replace 
of the `MotorBike` class. Meaning we need an adapter class. And this is where the Adapter Pattern comes into play. 
Let's grasp it using images:

![Adapter Pattern - Existing code and adaptee](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-existing-code-and-adaptee.png)

In our context, the Existing Code block may represent the client code. Therefore, our `start()` function. And the 
Adaptee may represent the `RoadBike` class.

Look at the blocks on the image above. Both have different shapes. They are not matching with each other. Because they 
have different shapes/interfaces. So we need an adapter to make them match. See the following images:

![Adapter Pattern - Existing code, new code, and adaptee](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-existing-code-new-code-and-adaptee-1.png)
![Adapter Pattern - Existing code, new code, and adaptee](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-existing-code-new-code-and-adaptee-2.png)

The New Code may represent the Adapter class. It makes the Existing Code and the Adaptee match.

So we can now understand that we need a middleman. Therefore, an adapter class that can make the existing code and 
the new vendor class connect together. We will create the adapter class to solve the problem. But before doing that 
we will see the definition of the Adapter pattern.

## The Definition
The Adapter Pattern is a software design pattern. It has been categorized into structural design patterns by the Gang 
of Four (GoF). Because it helps structure or make your code modular. Let us see the definition the GoF have provided 
in their book:

> Convert the interface of a class into another interface clients expect. Adapter lets classes work together that 
> couldn't otherwise because of incompatible interfaces.

The adapter pattern lets classes work together. Therefore, it helps `RoadBike` and `MotorBike` classes work together. 
Meaning the `RoadBike` class will work where `MotorBike` works. But how to do it? Converting the interface of the 
`RoadBike` class into the interface that the `MotorBike` class has or has implemented. When to do so? When our client 
expects.

Because our client code the `start()` function is expecting objects created from the `RoadBike` class as the project 
manager said. But the `RoadBike` class is not compatible with the `VehicleInterface`. So we need to apply the Adapter 
Pattern to solve the problem.

*Point to be noted that here the interface does NOT mean the language feature ONLY. It could be a concrete class too. The interface may mean the behaviors or structure of a class or what a class can do.*

If we apply the Adapter Pattern to the problem, then that would change the interface of the `RoadBike` class in such 
a way that the client code expects. To do so, we need an adapter class.

## Adapter The Middleman
It is time to implement the adapter class. Let's create a class that acts as a middleman between the existing code and 
the new vendor class to make them work together. There are two ways to define an Adapter class: the Class Adapter and 
the Object Adapter. At first, we will see how to implement an Object Adapter.

**Object Adapter**: The object adapter uses object composition to implement the Adapter class. The object composition 
is a kind of object association. Therefore, the way we associate an object to a class we are working on.

![Adapter Pattern - UML for Object Adapter](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-uml-for-object-adapter.png)

Here the Client is our `start()` function. The Target is our `VehicleInterface` and the Adaptee is our new vendor 
class  -  the `RoadBike` class.

In object composition, we use an instance of an object of the adaptee (the new vendor class) in the adapter class. 
Therefore, we will use an instance of the `RoadBike` class in the adapter class. Then we delegate the method calls to 
the object created from the `RoadBike` class to conform to the `VehicleInterface`. Check the following adapter class:

```php
<?php

class RoadBikeAdapter implements VehicleInterface
{
    private $roadBike;

    public function __construct()
    {
        $this->roadBike = new RoadBike();
    }

    public function startEngine()
    {
        $this->roadBike->padal();
    }

    public function speed()
    {
        $this->roadBike->speed();
    }
}
```

As of now, the `RoadBikeAdapter` class is implementing the `VehicleInterface` so we can use this class in our client 
code. See below:

```php
<?php

$roadBikeAdapter = new RoadBikeAdapter();

start($bicycleAdapter);

// Outputs

The pedaling has been started...

Setting the speed to 20 km per hour.
```
Keep in mind we could have used [Object Aggregation](https://medium.com/r/?url=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F885937%2Fwhat-is-the-difference-between-association-aggregation-and-composition) 
instead of object composition to implement object adapter. An Object Aggregation is another kind of object association. 
I will add an adapter class example using aggregation at the very bottom of this article.

**Class Adapter**: Uses inheritance to implement the Adapter class.

![Adapter Pattern - UML for Class Adapter](https://raw.githubusercontent.com/unclexo/software-craftsman/main/assets/design-patterns/adapter-pattern/adapter-pattern-uml-for-class-adapter.png)

To create an adapter class using inheritance, we need to inherit from the Adaptee. We already know the adaptee is 
the `RoadBike` class. Let's create the adapter class by extending the `RoadBike` class:

```php
<?php

class RoadBikeAdapter extends RoadBike implements VehicleInterface
{
    public function startEngine()
    {
        $this->padal();
    }

    public function speed()
    {
        $this->speed();
    }
}
```
It is simple huh? Yes, it is. Let's use it in the client code. It outputs the same as before:

```php
<?php

$roadBikeAdapter = new RoadBikeAdapter();

start($roadBikeAdapter);

// Outputs

The pedaling has been started...

Setting the speed to 20 km per hour.
```

Implementing an adapter class using inheritance is not suggested, rather it is suggested follow the software design 
principle - [Favor composition over inheritance](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FComposition_over_inheritance).
Because inheritance occurs at the compile time while object composition happens at runtime. At runtime we can 
decide which class to use maybe based on an argument meaning we have options to choose. But having used inheritance, 
we need to stick to only one subclass.

Otherwise, this is a simple example of an adapter pattern using inheritance. **What if there is more than one adaptee.** 
Then we would NOT be able to create the Class Adapter using inheritance. Because languages like PHP, JAVA, C# etc. do not 
support multiple inheritance. But it is possible by the languages that support multiple inheritance, for example, 
Python, C++ etc.

An object adapter class example using object aggregation:

```php
<?php

class BicycleAdapter implements VehicleInterface
{
    private $bicycle;

    public function __construct(BicycleInterface $bicycle)
    {
        $this->bicycle = $bicycle;
    }

    public function startEngine()
    {
        $this->bicycle->padal();
    }

    public function speed()
    {
        $this->bicycle->speed();
    }
}
```

Have you noticed anything? The `BicycleAdapter` class is now conforming to the "Program to interface, not an 
implementation" principle. See the constructor of it is taking the `BicycleInterface` interface as a parameter and 
acting as a polymorphic parameter. Because of this parameter it can take any bicycle object created from a class that 
implements the `BicycleInterface` interface. So the Polymorphism has also been applied here.

Again the `BicycleAdapter` class is also conforming to the [Open-closed Principle](https://medium.com/@unclexo/ocp-the-open-closed-principle-33eab31c7b92). 
How? Notice if you want to pass any bicyle object other than RoadBike object, you do NOT have to modify the 
BicycleAdapter class.

That's it.

## Reference:
* [Head First Design Patterns](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FHead-First-Design-Patterns-Brain-Friendly-ebook%2Fdp%2FB00AA36RZY)
* [Agile Software Development, Principles, Patterns, and Practices](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FSoftware-Development-Principles-Patterns-Practices%2Fdp%2F0135974445)
* [Design Patterns: Elements of Reusable Object-Oriented Software](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FDesign-Patterns-Elements-Reusable-Object-Oriented%2Fdp%2F0201633612)