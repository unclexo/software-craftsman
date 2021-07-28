# DIP: Dependency Inversion Principle

DIP stands for Dependency Inversion Principle. It is one of the principles of Object-Oriented Design. Robert Cecil Martin¹ (widely known as Uncle Bob) introduces a number of principles in his famous book:

[Agile Software Development, Principles, Patterns, and Practices](https://medium.com/r/?url=https%3A%2F%2Fwww.amazon.com%2FSoftware-Development-Principles-Patterns-Practices%2Fdp%2F0135974445)

He describes five OOD principles in this book. They are collectively known as the SOLID Principles. These principles have received wide attention in the software industry. The Dependency Inversion Principle is the last one of the five principles.

Uncle Bob gives a definition of the Dependency Inversion Principle in his book. The definition is as the following:

> A. High-level modules should not depend on low-level modules. Both should depend on abstractions.

> B. Abstractions should not depend on details. Details should depend on abstractions.

Eric Freeman and Elisabeth Robson, in their Head First Design Patterns book, reference this definition in a very short and concise form. They say:

> Depend upon abstractions. Do not depend upon concrete classes.

You may guess somewhat what the DIP is about. It says high-level modules or classes should NOT depend on concrete classes (low-level classes). Again, it says high-level and low-level classes both should depend on the abstractions. 

> Concrete classes are classes that have implementation details.

These abstractions should not depend on implementation details, rather, implementation details should depend on abstractions.

## High-level Modules or Classes
First, we will see what high-level modules are. The high-level modules are Client classes. What are the Client classes? In our case, Clients are classes that have dependencies   - the high-level modules!

Then, what are low-level modules? The dependent classes are low-level modules or classes.
 
> Do not mess up with Dependency Injection (DI). DIP is completely different from DI. DIP may have a similarity, in terms of appearance, with DI but they are totally different from each other in terms of purpose.

Not understood yet? Let us see a code example to make it clear.

```php
<?php

class SocialPoster
{
    protected $twitter;
    
    // Notice the "Twitter" dependent class here
    public function __construct(Twitter $twitter)
    {
        $this->twitter = $twitter;
    }
    
    public function post($data)
    {
        return $this->twitter->post($data);
    }
}
```

In the code above, `SocialPoster` is the high-level module/class and Twitter is the low-level module/class. But the DIP says high-level classes should not depend on low-level classes. But we have done so in the code above. What is the problem with it then?

Can you find anything in the code that seems troublesome to you? No? Check out below:

* As Twitter is a concrete class, so it is heavily coupled with the `SocialPoster` class. It violates the loose coupling principle.

* Any changes to the Twitter class will make the SocialPoster class change. It breaks the encapsulation pillar of OOP.

* The `SocialPoster` class is not reusable. This hits one of the advantages of OOP (Reusability).

* The `SocialPoster`  class is limited to post on Twitter social media. It could have made the $socialMedia parameter polymorphic, thus, this breaks the polymorphism pillar of OOP.

and more.

So what is the solution? Well, the solution is somewhat tricky. What we have to do is to invert the dependency. How would we invert the dependency?

## Inversion of Dependency 
Generally, when we start learning OOP, we use concrete classes as the dependent classes when a class depends on them. But we have seen dealing with a concrete class as a dependent class has produced some problems. We need to find out a way so that the `SocialPoster` class becomes loosely coupled and does NOT depend on any concrete class. How? We can do that inverting dependency!

To invert the dependency, we have to make sure our high-level classes use abstractions. What are abstractions? Abstractions are concepts (not detail) or contracts or rules or laws. The Contracts or Rules or Laws tell what to do. But, keep in mind, they do NOT tell you how to do it. That is why they are called abstractions.

Let us find out why the high-level class `SocialPoster` needs a dependency on low-level class Twitter. There could have many reasons for using the Twitter class. But, in our case, the main reason is to post on social media.

Let us abstract posting on social media. We can invert the dependency using either an abstract interface or an abstract class. That means we will pass an abstract interface to the constructor of the Client class SocialPoster instead of a concrete class. Thus depending on abstraction, not on a concrete class, the SocialPoster class inverts managing its dependency tradition.

So make a contract/rule/law.

```php
<?php

interface SocialPosterInterface
{   
    public function post($data);    
}
```

This is our abstraction for posting on social media. Abstraction provides such a law that dictates what you have to abide by. In the code above, the abstraction, meaning the contract/rule/law dictates it has a method `SocialPosterInterface::post($data)` that can be used to post on social media (instead of using a method from a concrete class).

Now we can make the `SocialPoster` class depend on the abstraction  - the `SocialPosterInterface`.

```php
<?php

class SocialPoster 
{
    protected $socialMedia;
    
    public function __construct(SocialPosterInterface $socialMedia) 
    {
        $this->socialMedia = $socialMedia;
    }
    
    public function post($data) 
    {
        return $this->socialMedia->post($data);
    }
}
```

As you can see the `SocialPoster` class manages its dependency based on abstraction, so there is no need to depend on low-level classes. No matter how the `post($data)` method of `SocialPosterInterface`  is implemented. It just has the knowledge about the contract or rule or law but not about the implementation details like before. The `SocialPoster` class is no more coupled with a limited API. It has now unlimited power. Because, it can receive any kind of social media API object as an argument, even if the one has not been invented yet. So it is tremendously scalable.

The `$socialMedia` parameter of SocialPoster class has become polymorphic because of using abstraction. You can use any kind of object as an argument that implements the `SocialPosterInterface`.

## Low-level Modules/Classes
The `SocialPoster` class is now decoupled in the sense that if you change in the low-level class (for example, `Twitter` class) does not affect the `SocialPoster` class anymore. Moreover, it can be reused in any context where low-level modules conform to the SocialPosterInterface abstract interface.

Remember abstraction provides one or more concepts that tell you what to do but not how to do it. The how-to-do-it will be implemented by the low-level classes. So whatever needs to be changed in low-level classes will not affect the high-level `SocialPoster` class unless low-level classes break the rule provided by that abstract interface.

As we have abstracted posting on social media we can now create as many low-level classes as we need conforming to `SocialPosterInterface` abstract interface.

```php
<?php

class Twitter implements SocialPosterInterface
{
    // More properties and methods
    
    public function post($data) 
    {
        var_dump("Posting: " . join(',', $data));
    }
}

class Facebook implements SocialPosterInterface
{
    // More properties and methods
    
    public function post($data) 
    {
        var_dump("Posting: " . join(',', $data));
    }
}
```

Now the abstraction is not depending on details, rather, details are depending on abstraction. Note `Twitter` or `Facebook` low-level class depends on `SocialPosterInterface` but `SocialPosterInterface` (abstraction) does not depend on `Twitter` or `Facebook` class (implementation details).

You can create more low-level classes. Just abide by the contract/rule/law. It will fit in the high-level `SocialPoster` class. You can now see the power of DIP.

Let the high-level SocialPoster class call the social media it needs.

```php
<?php

$media = [new Twitter, new Facebook];

$data = [
 'userId' => 23,
 'content' => 'Dependency Inversion Principle',
];

// Post on all social media
foreach ($media as $medium) {
    $poster = new SocialPoster($medium);
    $poster->post($data);
}
```

## Inversion of Ownership
Have you noticed anything? Something has happened after implementing DIP. And that is an inversion of ownership. Ownership of what? Ownership of the interface. Who owns the interface?
After applying the DIP, it seems that the SocialPoster owns the ownership of the `SocialPosterInterface `. But we have seen Twitter and Facebook both use SocialPosterInterface. The ownership of the `SocialPosterInterface` has been ambiguous. Should we put it in the high-level module or the low-level module? Does `SocialPoster` own it? Or does `Twitter` or `Facebook` own it?

But why is this important?

> Interface needs to stand alone without belonging to either group. - by Uncle Bob

Because the interface can be used by lots of different clients and implemented by lots of different modules. That is why we need to take care of the inversion of the interface ownership.

DIP has made it happen the inversion of interface ownership. So this is a problem. We can solve it by providing a very generic name to the interface so that, that does not imply the use of any high-level and low-level module together.
If we change the name of `SocialPosterInterface` to `PostInterface` then it may be kept in a separate module. It may be used later by different modules. The `PostInterface` abstract interface is NOT now limited to be used for Social Postings. It may be used for Blog postings, Forum postings, and maybe for others.

```php
<?php

interface PostInterface
{
    /**
     * Publishes post
     *
     * @param array $data
     * @return mixed
     */
    public function post($data);    
}
```

So our high-level class SocialPoster should be looked like the below.

```php
<?php

class SocialPoster 
{
    protected $socialMedia;
    
    public function __construct(PostInterface $socialMedia) 
    {
        $this->socialMedia = $socialMedia;
    }
    
    public function post($data) 
    {
        return $this->socialMedia->post($data);
    }
}
```

## Conclusion
DIP is one of the most powerful OO design principles. It produces reusable code. It reduces code duplication. It helps you write the least code. Applying DIP makes code testable and scalable. DIP allows you to write code that can be compatible with the code that will be written in the future.

Leaning DIP, you would be able to write robust code and easily deal with more design principles such as Design by Contract (DBC), Do NOT Repeat Yourself (DRY), Open-Closed Principle (OCP), Loose Coupling or Decoupling, and maybe more. And you will learn how to write polymorphic code.

[**Download working code**](https://medium.com/r/?url=https%3A%2F%2Fgist.github.com%2Funclexo%2Fc82452fe3dced63756383e8e6261043e)

## References:
* [Robert Cecil Martin (Uncle Bob)]()

* [Head First Design Patterns]()

* [Agile Software Development, Principles, Patterns, and Practices]()