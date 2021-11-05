![The Factory Method Pattern - Cover Photo](https://raw.githubusercontent.com/unclexo/articles/main/assets/design-patterns/factory-method/the-factory-method-pattern-cover-photo.jpg)

# Factory method -  Dealing with object creation and conditional switch or if-else nested statements

In this article, we will discuss the Factory Method design pattern. We will know why we need the design pattern, how the pattern solves specific design problems, how the pattern helps establish encapsulation.

### The Problem
We will start with the problem. The problem we will discuss is the switch or if-else nested statements. Let us say, we have an if-else nested statement that **creates objects** in the client code.

Is there any developer who has NOT dealt with if-else or switch nested statements? Be it small or large! I think the answer is NO. Every developer has used it. 

We will use the concept of the MVC design pattern and fake managing payments via payment gateways to take this article further. You may choose other examples.

An application that applies the MVC design pattern, generally, sends the user's requests to MVC's C, therefore, the controller's method. Let me make it clear. For example, our app supports payment gateways. If a user buys something on our app, he/she will pay us via a payment gateway. To do that he/she will go through a payment gateway, say, Stripe or Paypal.

He/she will provide the amount, card details, etc. With this information, finally, he/she will submit a request to the server. The server will take the request and pass it to our app and our app will send it to a controller's method. By the way, what would it look like? See the code below:

```php
<?php

class PaymentController 
{  
    public function handle(Request $request) 
    {
        $config = $this->container->get('config');
        $paymentType = $request->get('paymentType');
        
        $response = null;
        
        if ('paypal' == $paymentType) {
        
            $paypal = new Paypal();
            $paypal->setApiKey($config['paypal']['api_key']);
            
            // Other setups
            
            $response = $paypal->send();
        
        } elseif ('stripe' == $paymentType) {
        
            $stripe = new Stripe();
            $stripe->setApiKey($config['paypal']['api_key']);
            
            // Other setups
            
            $response = $stripe->send();
        
        } elseif ('other' == $paymentType)  {
          
            // And goes on
        }
        
        // Now do whatever you need to do using "$response"
    }
}
```

Note: The spaces are for maximum clarity. It's just for demonstration purposes.

This is the kind of approach amateur developers do. And the software firms that do NOT care about writing well-designed code (Much I have seen them).
 
**So what is wrong with that?**

Problem 1: This approach has problems when we use it in our client code. Why? The client code that uses the if-else nested statements for creating payment gateway objects deals with concrete classes  - `new Paypal()`, `new Stripe()`, etc. So it tightly couples those objects to the client code  -  the `PaymentController` class. It violates software design principles. For example, **Loose coupling**, **Program to an interface, not an implementation**, etc.

Problem 2: The client code is not encapsulated. Why? Because it is your eternal old friend **CHANGE**. What if the concrete classes' APIs are changed or you need to add a new payment gateway? You then have to modify your client code.

Problem 3: Again it breaks the **Open-closed principle**. What if you want to add another payment gateway? You then need to extend the if-else nested statements. That means the client code is not "closed for modification". Thus it breaks OCP.

Problem 4: The object creation code is not reusable. If you reuse this code, you will end up putting the if-else nested statements in many places in our app. But doing so violates another design principle: **Don't Repeat Yourself**. If the implementation of those concrete classes changes, you then have to change all the places where you have used if-else nested statements.

Problem 5: Otherwise, it will grow the conditional statement to an uncertain degree. But "The ﬁrst rule of functions is that they should be small. The second rule of functions is that they should be smaller than that." This rule is also broken here by the handle() function.

**So what is the solution?** 

The solution is to separate the object creation by moving the if-else nested statements to another place. This is where the Factory Method pattern comes into play. This is where the Factory Method pattern helps us move the object creation in a separate class and lets us modify code in one place. 

> Separate the if-else nested statements that handle object creation from the client code.

![Factory Method: The client code for managing payment gateways](https://raw.githubusercontent.com/unclexo/articles/main/assets/design-patterns/factory-method/the-client-code-for-managing-payment-gateways.png)

### The Definition
The Factory Method is a software design pattern. It has been categorized into creational design patterns by the Gang of Four (GoF). Because its job is to create objects. Let us see the definition the GoF have provided in their book:

> Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

If you understand anything from the definition, that will be great. No problem if you do NOT. I am going to break the definition into pieces so that we can understand it clearly.

Point 1: The design pattern defines an interface for creating an object.

Point 2: The design pattern lets subclasses decide which class to instantiate.

Point 3: Finally, what it does, as a whole, is it lets a class defer the instantiation to subclasses.

We will tackle each point to implement the Factory Method. Then we will see how the Factory Method helps move the if-else nested statements from the client code, separate object creation, and design better software.

## Implementation Of Factory Method

**Point 1: Defining an interface for creating an object**

Point 1 says it provides an interface for creating an object. As we are working on the payment gateways, we should have an interface for creating payment gateway objects instead of creating them in the client code using if-else nested statements. So that they do not couple with the client code, the `handle()` method of `PaymentController` method. Let us provide an interface for doing so.

The point to be noted is that if you are using either **interface** or **abstract method/class**,  you are abstracting something (in our case, we are abstracting object creation). Both do the same thing - abstraction. Put it in your brain.

We will be moving all the responsibility for instantiating payment gateways into a method that acts as a factory. This method is known as the **factory method**.

```php
<?php

abstract protected function createPaymentGateway(Request $request);
```

Let us move the object creation part from the client code to the interface that will be responsible for creating payment gateway objects.

![Factory Mehtod: Moving the object creation code to an abstract method](https://raw.githubusercontent.com/unclexo/articles/main/assets/design-patterns/factory-method/moving-object-creation-code-to-an-abstract-method.png)

As `createPaymentGateway()` is an abstract method so we need an abstract class. See the code snippet below:

```php
<?php

abstract class AbstractPaymentGateway 
{    
    public function process(Request $request)
    {
        $paymentGateway = $this->createPaymentGateway($request);
        $response = $paymentGateway->send();
        
        return $response;
    }
    
    abstract protected function createPaymentGateway(Request $request);
}
```

Here `AbstractPaymentGateway` is an abstract class that provides the abstraction for creating objects. By using abstraction, we make a contract. The contract is for creating payment gateway objects. The job of object creation has been ordered to move to subclasses. Now a subclass must implement the contract to create payment gateway objects. This meets Point 1. Meaning we have now an interface for creating objects.

**Point 2: Letting subclasses decide which class to instantiate**

Notice the `process()` method above. It uses the `createPaymentGateway()` factory method to get a payment gateway object to process the payment. But to get the payment gateway object, the `AbstractPaymentGateway` abstract class depends on its subclasses. Since we abstract the object creation.

Let us create a subclass `PaymentGateway` that extends the abstract class `AbstractPaymentGateway`. See what it looks like below:

```php
<?php

class PaymentGateway extends AbstractPaymentGateway
{    
    protected function createPaymentGateway(Request $request)
    {
        $config = $this->container->get('config');
        $paymentType = $request->get('paymentType');
        
        $paymentGateway = null;
        
        if ('paypal' == $paymentType) {
        
            $paymentGateway = new Paypal();
            $paymentGateway->setApiKey($config['paypal']['api_key']);
            
            // Other setups
        
        } elseif ('stripe' == $paymentType) {
        
            $paymentGateway = new Stripe();
            $paymentGateway->setApiKey($config['paypal']['api_key']);
            
            // Other setups
        
        } elseif ('other' == $paymentType)  {
          
            // And goes on
        }
        
        return $paymentGateway;
    }
}
```

The `AbstractPaymentGateway` abstract class lets the `PaymentGateway` subclass give the responsibility to make payment gateway objects. The `PaymentGateway` subclass implements `createPaymentGateway()` method for creating payment gateway objects. By doing so, the `AbstractPaymentGateway` abstract class does what Point 2 says to do. Therefore, it lets the PaymentGateway subclass decide which payment gateway object to instantiate.

When Point 1 and Point 2 are implemented then Point 3 is automatically applied. The object creation is deferred when you let subclasses decide it.

We implemented the Factory Method pattern. Cheers!

### What We Have Done
We have applied the Factory Method pattern to tackle the object creation using the if-else nested statements in the client code. The Factory Method abstracts the object creation and by using abstraction, it ensures the object creation must be done/implemented by a subclass.

We have made the abstraction via the `createPaymentGateway()` abstract method of the `AbstractPaymentGateway` abstract class. We have moved the if-else nested statements from the client code  to a factory method `createPaymentGateway()` implemented by the `PaymentGateway` class.

By doing so, the Factory Method helps move the object creation part in one place. We have encapsulated object creation in one separate class. We now have only one place to make modifications when the implementations of the payment gateway classes change.

### The Question
Now the question is, have we solved all the problems that occur by using if-else nested statements in the client code? Let us pause the answer here. First, we will see how the code looks like after applying the Factory Method:

```php
<?php

class PaymentController 
{
    protected $paymentGateway;
    
    public function __construct(AbstractPaymentGateway $paymentGateway) 
    {
        $this->paymentGateway = $paymentGateway;
    }
    
    public function handle(Request $request) 
    {
        $response = $this->paymentGateway->process($request);

        if ($response->isSuccessful()) {

            // Payment was successful
            print_r($response);

        } else {

            // Payment failed
            echo $response->getMessage();
        }

        // Other stuff
    }
}
```

Now, let's start with the constructor method of the new `PaymentController` class. It uses the `AbstractPaymentGateway` abstract class as a parameter. This parameter is acting as a polymorphic parameter. Because any implementation  of any subclass of `AbstractPaymentGateway` abstract class can be used here. So if the `PaymentGateway` implementation does not work somehow, we can use another subclass instead.

Now, let's come to the client code the `handle()` method of the new `PaymentController` class. Check out the line 14. The `handle()` method just calls the `process()` method on `AbstractPaymentGateway` and handles the response it gets from the payment gateway. Yes, it is that simple.

Setting up card details, purchase amount, currency, etc. will be managed by the subclass of `AbstractPaymentGateway`. In our case, it is the `PaymentGateway` subclass.

(Here I am assuming the Request object provides data the way the payment gateway expects. So they should be in a contract.)

### The Proof
Now it is time to see whether the problems have been solved after applying the Factory Method. Let us verify one by one.

Proof to problem 1: As the object creations of the payment gateway classes have been moved to another class, so the payment gateway objects do not couple with the client code anymore. So it meets "Loose Coupling".

Notice the `AbstractPaymentGateway` parameter of the constructor of the `PaymentController`. As it is an abstraction for creating payment gateway objects, so it can behave polymorphically. And we can pass any object to the constructor created from any subclass of the AbstractPaymentGateway abstract class. Thus we implement "Program to an interface, not an implementation".

Proof to problem 2: If any change happens in the concrete classes of payment gateway objects then that will not affect the client codeanymore. Because we have encapsulated the object creation to another class. This applies encapsulation.

Proof to problem 3: If we want to add a new payment gateway object, we can do that without modifying the client code. So the client code is now "closed for modification". This conforms to OCP.

Proof to problem 4: The object creation job is now encapsulated in one class. We can reuse the `PaymentGateway` subclass wherever we need it in our app. Now the object creation code is reusable.

Proof to problem 5: The client code, the `handle()` method of `PaymentGateway` class has now less than 20 lines of code (including vertical spaces). This solves Problem 5- A function should be small.

### Using Interface
What if we wanted to use an interface instead of an abstract class? Nothing! But a small difference - an abstract class can have method implementations (as it may define common/basic behaviors/rules that child classes need) where an interface can NOT. But basically, both do the same thing - abstraction. I have said this before. This time we would abstract object creation using an interface.

```php
<?php

interface PaymentGatewayInterface 
{
    public function createPaymentGateway(Request $request);
}
```

And the implementation of the `PaymentGatewayInterface` interface is as below:

```php
<?php

class PaymentGateway implements PaymentGatewayInterface
{    
    protected function createPaymentGateway(Request $request)
    {
        // The method body is as same as the previous one
    }
}
```

And the `PaymentController` would look like the below:

```php
<?php

class PaymentController 
{
    protected $paymentGateway;
    
    public function __construct(PaymentGatewayInterface $paymentGateway) 
    {
        $this->paymentGateway = $paymentGateway;
    }
    
    // The rest is as same as the previous one
}
```

### Conclusion
What we have learned, in this article, is how to apply the Factory Method pattern to specific problems -separating object creation and moving if-else nested statements from client code. 

Using the if-else nested statements for creating objects in the client code breaks several  software design principles like Loose Coupling, OCP, DRY, etc. To solve this problem we apply the Factory Method design pattern.  It helps move the if-else nested statements to a separate class. By doing this, we make sure only one place to make modifications when the implementation changes.

Follow me here on Github, [Twitter](https://twitter.com/unclexo), [Medium](https://medium.com/@unclexo), and [LinkedIn](https://www.linkedin.com/in/unclexo)

### References
1. [Loose Coupling](https://en.wikipedia.org/wiki/Loose_coupling)
2. [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
3. [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
4. [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
5. [Head First Design Patterns: A Brain-Friendly Guide](https://www.amazon.com/Head-First-Design-Patterns-Brain-Friendly/dp/0596007124)
6. [Agile Software Development Principles, Patterns, and Practices](https://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445)