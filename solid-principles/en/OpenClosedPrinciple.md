![OCP: The Open-Closed Principle](https://raw.githubusercontent.com/unclexo/articles/main/assets/solid/open-closed-principle.jpg)

# OCP: The Open-Closed Principle
OCP stands for Open-Closed Principle. It is one of the principles of Object-Oriented Design. Robert Cecil Martin¹ (widely known as Uncle Bob) introduces a number of principles in his famous book:

[Agile Software Development, Principles, Patterns, and Practices](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FSoftware-Development-Principles-Patterns-Practices%2Fdp%2F0135974445)

He described five OOD principles in this book. They are collectively known as the SOLID Principles. These principles have received wide attention in the software industry. The Open-Closed Principle is the second one of the five principles. Otherwise, O of SOLID.

This principle was first coined by Bertrand Meyer² in 1988. He gave a short definition of the principle in his book³:

> Modules should be both open and closed.

But later, Uncle Bob, in his book, referred to this definition in the following way while describing the Open-Closed Principle:

> Software entities ( classes, modules, functions, etc. ) should be open for extension, but closed for modification.

The definition says you should not modify the source code shipped with the package rather it should have a way so that you can add new functionalities. This definition tells us two rules to establish onto a class/module/function that we are building:

**Open for Extension**: A class can be extended according to requirements. You can then add new functionalities. You should design a class in such a way so that it has the option to add new functionalities. In other words, you can change what the class does.

**Closed for Modification**: A class can not be modified. This means you should not modify source code to add new behaviors. You can not change its already available functionalities to add new functionalities that already work.

First, we would violate the OCP through examples. Then we would see how we can satisfy the OCP. This is actually a better way to understand how a design principle works.

## Violating the OCP!

Let us think of we are going to build a simple online store. We would sell books there. We need to show information about each book. To do so, we need a `Book` class. Let us make it like the following example:

```php
<?php

class Book {
  
  protected $title;
  protected $author;
  
  public function getTitle() {
    return $this->title;
  }
  
  public function setTitle($title) {
    return $this->title = $title;
  }
  
  public function getAuthor() {
    return $this->author;
  }
  
  public function setAuthor($author) {
    return $this->author = $author;
  }
  
  public function display() {
    var_dump($this->getTitle() . ' written by ' . $this->getAuthor());
  }
}
```

Now imagine, our online store has been making good profits. So we decide to add more products in our store. We want to add Movies, Songs or Paintings to our online store. How would we do that? OK. There are several ways we can attempt to do that.

**First**, we can attempt to use the `Book` class for displaying information about movie. To do so, we need to add `getDirector()` and `setDirector()` methods to the `Book` class . Moreover, we have to modify the `display()` method of the `Book` class as the following:

```php
<?php

class Book {
  
  // other properties
  
  private $director;
  
  // other methods
  
  public function getDirector() {
    return $this->director;
  }
  
  public function setDirector($director) {
    return $this->director = $director;
  }
  
  public function display($type) {
    if ('book' == $type) {
      var_dump($this->getTitle() . ' written by ' . $this->getAuthor());
    } elseif ('movie' == $type) {
      var_dump($this->getTitle() . ' directed by ' . $this->getDirector());
    }
  }
}
```

For the sake of simplicity, we omitted some properties and methods.

To display information about movie, we needed to modify `Book` class - adding two more methods and modifying `display()` method. This means we went against what the second rule “Closed for Modification” section says. We violated one of the two rules. Thus we can not modify the `Book` class (the existing code).

**Second**, we could have extended the `Book` class by `Movie` class to implement the `display()` method to display movie information. But we can not be able to extend the `Book` class because a movie is not a book. Even if we do so that would break another principle called LSP.

**Third**, what if we create a class for each of them? Well, there would be another problem. Each class will then have `getTitle()` and `setTitle()` methods for several times that make code duplication. At this point, you may guess that the `Book` class is not open for extension too. It seems we are stuck here.

So there is no proper way to add new products to our shop with our existing code. Thereby the class is not open for extension and closed for modification. The design of `Book` class violates the OCP therefore.

How is it possible that the new behaviors of a class can be achieved without changing its source code? The answer is abstraction.

## Abstractions
Abstractions are abstract base classes where the unbounded group of possible behaviors is defined for their derivative classes. For example, in an abstract class, you define methods that can be used in the child classes for playing similar roles with different implementations. Basically, an abstract class defines the basic structure of its child classes. So, it is possible for a class to manipulate an abstraction.

And such a class can be closed for modification because it depends on the abstraction that is fixed. But the behavior of that class or module can be extended by creating new subclasses based on the abstraction.

In our case, we would follow a simple algorithm for products. We would display title and creator for a product. We need a `display()` method for showing information about the product. Let us make an abstraction for displaying product information:

```php
<?php

abstract class AbstractProduct {
  
  protected $title;
  
  public function getTitle() {
    return $this->title;
  }
  
  public function setTitle($title) {
    $this->title = $title;
  } 
  
  /**
   * Displays information about a product
   */ 
  abstract protected function display();
}
```

We make an abstract class named `AbstractProduct`. Notice we put an `abstract` keyword in front of the class and its `display()` method to make them more self-descriptive. Languages like Java, C#, PHP etc. have this keyword to make abstract classes and methods.

`AbstractProduct` is now such a class that is now conforming to OCP. How? Try to remember what the abstraction and the OCP describe above. This class is now open for extension and closed for modification. Because the abstraction is fixed and if we need to add more behaviors it will allow us to do by creating subclasses. How? Check out the following two code examples.

Now let us create our `Book` class depending on the abstraction:

```php
<?php

// Book Class
class Book extends AbstractProduct 
{
  protected $author;
  
  public function getAuthor() { 
    return $this->author;
  }
    
  public function setAuthor(string $author) 
  {
    $this->author = $author;   
  }
    
  public function display() {
    var_dump($this->getTitle() . ' written by ' . $this->getAuthor());
  }
}
```

A book is a product so we can extend `AbstractProduct`. Then we added an accessor and a mutator methods to `Book` class to be used inside `display()` method to show information about books.

Now if we want to display information about a movie, that would be a new product. To do so we do not have to modify the `display()` method of the `Book` class anymore. We can depend on the abstraction. We can add that product to our shop by creating a new subclass.

Let us create a new product type called movie:

```php
<?php

// Movie Class
class Movie extends AbstractProduct 
{
  protected $director;
  
  public function getDirector() 
  {
    return $this->director;
  }
  
  public function setDirector(string $director) 
  {
    $this->director = $director;
  }
  
  public function display() {
    var_dump($this->getTitle() . ' directed by ' . $this->getDirector());
  }
}
```

A movie is also a product so we can extend `AbstractProduct`. Like `Book`, we added an accessor and a mutator methods to `Movie` class to be used inside `display()` method to show information about movies.

We are now able to add new behaviors depending on the abstraction. Using this approach we can now use `display()` method for different products or implementations. Therefore, we have been now able to close the base abstract class for modification. Thus we satisfy the OCP.

The method, we followed to conform to the OCP, is called **Template Method Pattern**⁴. That means we implemented this design pattern here to satisfy OCP. Gof has given a definition of this pattern in the book as follows:

> Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure.

In the example above, we made the skeleton in our `AbstractProduct` class. You may think of the `display()` method as the skeleton of an algorithm (displaying product information). Later, we redefined this method in different classes to display product specific information.

Interestingly, implementing Template Method Pattern, we implemented one more OOP principle called **Polymorphism⁵**. There is yet another way to satisfy OCP is called **Strategy Pattern**⁶. Gof has given a definition of this pattern in the book as follows:

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

We can satisfy the OCP by this design pattern too. But I am leaving Strategy Pattern implementation to satisfy OCP to you. I want you to dive into this pattern and satisfy the OCP using it.

## Applying OCP
Note that we kept things as simple as possible. Many things could have been added or removed from our code examples if we had built a real application. But the main theme would not be changed. I mean the OCP. Now we would see how the OCP can be useful to build robust, scalable and maintainable application. Check out below:

```php
<?php

class Product {
  
  private $product;
  
  public function __construct(AbstractProduct $product) {
    $this->product = $product;
  }
  
  public function info() {
    return $this->product->display();
  }
}

// Creates book object
$book = new Book();
$book->setTitle('PHP Objects, Patterns, and Practice');
$book->setAuthor('Matt Zandstra');

// Shows info about book
$product = new Product($book);
echo $product->info(); // 'PHP Objects, Patterns, and Practice by Matt Zandstra'

// Creates book object
$movie = new Movie();
$movie->setTitle('Forrest Gump');
$movie->setDirector('Robert Zemeckis');

// Show info about movie
$product = new Product($movie);
echo $product->info(); // 'Forrest Gump by Robert Zemeckis'
```

Here we created a class called `Product`. It has a method called `info()`. This is now responsible for delegating `display()` method for a product class, i.e. `Book`, `Movie`. The key point here is we associated `AbstractProduct` class to `Product` class so that this class can act as the **CLIENT** for derivative classes of `AbstractProduct` as the example above.

[Download working code example](https://gist.github.com/unclexo/43ade7c73d67d498a41d993d76ac3f3f)

### References:

1. [Robert Cecil Martin](https://en.wikipedia.org/wiki/Robert_C._Martin)
2. [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer)
3. [Object-Oriented Software 4. Construction](https://en.wikipedia.org/wiki/Object-Oriented_Software_Construction)
4. [Template Method Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)
5. [Polymorphism](https://en.wikipedia.org/wiki/Polymorphism_(computer_science))
6. [Strategy Pattern](https://en.wikipedia.org/wiki/Strategy_pattern)

