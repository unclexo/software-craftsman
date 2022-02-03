![OCP: The Open-Closed Principle](https://raw.githubusercontent.com/unclexo/articles/main/assets/solid/open-closed-principle.jpg)

# LSP: Liskov Substitution Principle

LSP stands for Liskov Substitution Principle. It is one of the principles of Object-Oriented Design. Robert Cecil Martin¹ (widely known as Uncle Bob) introduces a number of principles in his famous book:

[Agile Software Development, Principles, Patterns, and Practices](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FSoftware-Development-Principles-Patterns-Practices%2Fdp%2F0135974445)

He described five OOD principles in this book. They are collectively known as the SOLID Principles. These principles have received wide attention in the software industry. The Liskov Substitution Principle is the third one of the five principles. Otherwise, L of SOLID.

This principle was written by Barbara Liskov in 1988. She gives a definition of the principle in her book²:

>> What is wanted here is something like the following substitution property : If for each object O1 of type S there is an object O2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when O1 is substituted for O2 then S is a subtype of T.

But later, Uncle Bob paraphrases this definition in his book as follows:

>> Subtypes must be substitutable for their base types.

What does the definition mean? Say, we have a function called `factory(A)`. This function takes a reference of `A` as an argument where `A` is a class. `B` is the child class of `A`. As `B` is a derivative of `A`, we can pass in `B` to the `factory()` function as `factory(B)`. If passing `B` causes `factory()` function to misbehave, then `B` violates LSP.
Here `B` is a subtype of the base type `A`. Therefore, `B` subtype must be substitutable for the base type `A`. If not, that violates LSP. That is it.

Here is an abstract class named `Product`. We will use this class for describing the LSP. Just go through it. If you do not understand the code, no problem! I will explain in a few seconds.

```php
<?php

abstract class Product
{
  // other properties

  /**
   * Holds downloadable product's path
   *
   * @var string
   */
  protected $file;

  /**
   * Downloads downloadable product
   *
   * @throws \Exception
   */
  public function download()
  {
    if (! file_exists($this->file) && ! mime_content_type($this->file)) {
      throw new \Exception('The file provided was not downloadable.');
    }

    $filename = basename($this->file);

    // Sets headers for downloading file
    header('Expires: 0');
    header('Pragma: public');
    header('Cache-Control: must-revalidate');
    header('Content-Length: ' . filesize($this->file));
    header('Content-Type: application/octet-stream');
    header('Content-Disposition: attachment; filename="' . $filename . '"');

    readfile($this->file);
  }

  /**
   * Sets file to be downloaded
   *
   * @param string $file
   */
  public function setFile($file)
  {
      $this->file = $file;
  }

  // other methods
}
```

As you can see, this is an abstract class. So it is the basic structure of its subclasses. It has a method named `download()`. The method's doc block says, "Downloads downloadable product". If we looked at it carefully we would see, it would raise an exception if the file provided is not downloadable. And eventually, we see it has another method for setting a downloadable file path.

Now we would move towards LSP. So, we need a client function. That function will take an object of Product as an argument. We are doing so as the definition of LSP says above. This function will be used to download Audio, Video products for example. Let us see how the function looks like. By the way, you will get the working code attached below:

```php
<?php

// Our client function
function download(Product $product) {
  // Call the download method on product object
  $product->download();
}
```

Now according to LSP, if we pass any object that is created from the subclasses of `Product` class, it should work, otherwise, it would violate LSP. Let us create some subclasses for `Product` class.

```php
<?php

class Audio extends Product
{
  // Do other stuff
}

class Video extends Product
{
  // Do other stuff
}
```

Now we will create objects from those classes and pass in the `download()` function to check whether it can download or not.

```php
<?php

// Creates audio object
$audio = new Audio();
$audio->setFile('audio.mp3');


// Creates video object
$video = new Video();
$video->setFile('video.mp4');


// Calls download function passing different objects in it
download($audio);
download($video);
```

Wow! It works!. It can download the given file. So our `download()` function is LSP compatible meaning it is LSP compatible. Now we will go for violating LSP. Look at the following snippet of code:

```php
<?php
 
class Chocolate extends Product { 
  // Do other stuff
}

// Creates chocolate object
$chocolate = new Chocolate();
$chocolate->setFile('Chocolate can not be downloaded.');  

// Calls download function passing chocolate object in it
download($chocolate);
```

What is happening in the code above? As we know `Chocolate` is a product, so we could create a subclass from `Product` class. Because they have a **IS-A** relation between them. We then create the `Chocolate` object. We pass in the object to the `download()` function. But we are not able to download any chocolate, thereby, it throws an exception and says: "Please provide a downloadable file."

Please provide a downloadable file.
What this totally means is we violate LSP by passing in a `Chocolate` object to the `download()` function. Though `Chocolate` is a kind of product but it is not a downloadable product.

## Big Question!
Now the question is who is responsible for violating LSP? Is it the creator of the `download()` function? Is it the creator of `Product` class? Or is it the creator of `Chocolate` class?

One might argue it is the creator of `download()` function. Because `Chocolate` and Product has a **IS-A** relationship between them. There can have invariants of Product class that quite apply to the `Product` class.

Ok! Ok! But this is also true that the creator of `download()` function has every right to make the assumption that the `download()` function only maintains downloadable products. Now, what would you say? On the other hand, in general, the creator of `Chocolate` class did not violate the LSP. But it is who extends the `Chocolate` class from the `Product` class.

Notice, the `download()` method's doc block from `Product` class, it says this method will be used to download downloadable products only. So you have to pass in an object created from a subclass of `Product` as the argument to the `download()` function.

## Why LSP?
**LSP instructs how to properly inherit from classes**. Therefore, LSP tells us how to use or create derived classes the way they should be.

"What are the design rules that govern this particular use of inheritance? What are the characteristics of the best inheritance hierarchies? What are the traps that will cause us to create hierarchies that do not conform to the OCP?" These are the questions that are addressed by the LSP.

So when would you know that the design of a class or module is appropriate? How many miles exactly should we go for the design? Well, there are no **perfect** answers to these questions. Again, it is said that one must view the design in terms of reasonable assumptions made by users of that design.

Now a question comes up! Does anybody know what reasonable assumptions the users of that design are going to make? Those assumptions are not easy to anticipate. If you put all the assumptions with the design, that will then be the smell of **Needless Complexity**. This is not good to go. We do not want to load the design with lots of unnecessary abstractions. Then, what to do?

Well, **do not put all reasonable assumptions in your design but try to take care of common LSP violations**. And there is a way to provide some reasonable assumptions by following the design principle called *design by contract*.

## Design by Contract
We should not put all reasonable assumptions in our code. But there is a technique to make some reasonable assumptions explicit. The technique is called design by contract and is coined by Bertrand Meyer³.

Using DBC, the author of a class explicitly dictates the contract for that class. The contract is specified by declaring preconditions and postconditions for each method of a class. The preconditions and the postconditions are actually language features. These are not supported by PHP. But we can do that using doc block comments.

As we did with the `download()` method from Product class via the doc block. The doc block says the method can only download downloadable product and throws an exception if the product is not downloadable.

Now, for some reason, if the author needs different download logic, then he/she can do or implement the brand new logic in the subclasses. But to do that if the author implements less or more than what the base class's doc block says, it would then violate LSP.

## Common LSP Violations
There are some simple heuristics that can give us some clues about LSP violations. They are all about derivative classes.

**Degenerate Methods in Subclasses** - If a base class has a method, but a subclass from the base class does not need that method, then if the author of the subclass degenerates the method again, it will be a substitutable violation.

**Throwing Exceptions from Subclasses** - Another form of LSP violation is the addition of exception to the subclass while the base class does not expect that. Because then the base class can not be substituted by the subclass.

### References:
1. Robert Cecil Martin
2. Barbara Liskov
3. Bertrand Meyer

Download Code