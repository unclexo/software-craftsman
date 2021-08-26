![SRP: Single Responsibility Principle](https://raw.githubusercontent.com/unclexo/articles/main/assets/solid/single-responsibility-principle.jpg)

# SRP: Single Responsibility Principle

RP stands for Single Responsibility Principle. It is one of the principles of Object-Oriented Design. Robert Cecil Martin¹ (widely known as Uncle Bob) introduces a number of principles in his famous book:

[Agile Software Development, Principles, Patterns, and Practices](https://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445)

He describes five OOD principles in this book. They are collectively known as the SOLID Principles. These principles have received wide attention in the software industry. The Single Responsibility Principle is the first one of the five principles.

This principle was first described by Tom DeMarco² and Meilir Page-Jones³. They called it Cohesion⁴. Before knowing it, I want you to keep in mind:

> Software is all about behavior.

Now we need to be clear about Cohesion. What is this? Cohesion measures the strength of relationship between things that a module or a class contain. We know a class contains its methods (behaviors) and data. So Cohesion tells how strong or loose the relationship between methods and data is in a class.

A class or a module with high Cohesion is preferable. Because it makes code robust, reliable, reusable, scalable, maintainable and understandable. These are actually very desirable traits in making an application. In contrast, low Cohesion makes code difficult to maintain, test and understand.

Later, Uncle Bob in his book, shifts that meaning a bit and relates Cohesion to the forces that cause a module or a class to change. Now, we would explore SRP through the next sections.

## Implementation

Have you really kept in mind that software is all about behavior? Okay! Say, we have been asked to build a simple web app through which users can contact to the business owner. Here is what our contact app looks like:

```php
<?php

// Say, we would get data from a form submitted by a user
$data = [
  'firstName' => 'Jobaer',
  'lastName' => 'Sajeeb',
  'email' => 'unclexo@test.com',
  'message' => 'This is a test message'
];

// Creates a contact object
$contact = new Contact($data);

// Validates user data and sends email to the business owner
if ($contact->validate()) {
  
  // Sends email
  $contact->sendEmail();
  
  // We could do other tasks here
}
```
In the above script, we see an array of user data. Then we create a contact object using a class called Contact by passing user data in its constructor method. After that we call `validate()` method to validate user data and `sendEmail()` to send email.

This is what we need for our contact app.

Now we need to make the Contact class. According to the app, our Contact class will be as the following:

```php
<?php
/**
 * Handles contact data
 */
class Contact
{
  /**
   * Holds user data
   */
  protected $data;
  
  /**
   * Creates contact object
   * 
   * @param array $data
   */
  public function __construct(array $data = [])
  {
    $this->data = $data;
  }

  /**
   * Validates data
   */
  public function validate()
  {
    // write code for validating user data
  }
  
  /**
   * Sends email
   */
  public function sendEmail()
  {
    // write code for sending email
  }
}
```

Now we have our Contact class ready for our contact app. By the way, did you notice what behaviors does this class have? Can you figure them out? I know, you can. They are the methods of the Contact class. In the context of a class, methods are behaviors. Behaviors perform different tasks for the class. So we have three behaviors because we have three methods in our class. Keep them in mind.

With the Contact class in place, we would start looking into our Contact class in the context of Single Responsibility Principle. We need to involve ourselves to figure out the meaning of the word responsibility. Otherwise, R of SRP.

### What is our class responsible for?

Our Contact class has an attribute that holds user data. Then we have a constructor method where we initialize user data. Next, we have several methods. As we are dealing with user data, we must validate those data. Because users can put in some malicious data that we do not want to accept. So we have another method for data validation. We have yet another method for sending email.

Notice these **methods are responsible** for doing specific jobs. Now we would go through each and every responsibility. Check out below:

#### 1st Responsibility
The first responsibility of the class is to create contact object. Well, this is done by the constructor method. It also initializes user data. When we create contact object, behind the scene, the constructor method is called and initializes user data.

#### 2nd Responsibility
The second responsibility is to validate user data. The `validate()` method is responsible for validating user data. Look at the method below:

```php
<?php

/**
 * Validates user data
 */
public function validate()
{
  // Assumed validator API
  $validator = new Validator();

  $validator->set($this->data['firstName'], 'string|length:45|notEmpty');
  $validator->set($this->data['lastName'], 'string|length:45|notEmpty');
  $validator->set($this->data['email'], 'email|notEmpty');
  $validator->set($this->data['message'], 'string|length:200');

  if ($validator->validate()) {
    return true;
  }

  return false;
}
```

Here we are assuming we have a validation API. We are validating user data using this API. So the main responsibility of this method is to perform a validation job with the help of the validation API.

#### 3rd Responsibility
The third responsibility of the class is to send email. The `sendEmail()` method is responsible for sending an email. Look at the method below:

```php
<?php

/**
 * Sends email
 */
public function sendEmail() {

  // Assumed mail API
  $mail = new Mail();

  $mail->setFrom($this->data['email'], $this->data['lastName']);
  $mail->addTo('business.owner@example.com', 'Fakira');

  $mail->setSubject('From Contact App');
  $mail->setBody($this->data['message']);

  // Sends the email
  $mail->send();
}
```

Like validation API, we are assuming we also have a mail API for sending email.
Now we get the responsibilities of our class. Therefore, the Contact class has 3 responsibilities. Now we would see what is a responsibility in the context of SRP. Before looking at that, we need to take a look at the definition of SRP.

## The Definition
Uncle Bob gives a definition of the Single Responsibility Principle in his book. The definition is very simple and concise but very powerful. Here it is:

> A class should have only one reason to change.

The definition of SRP is very easy but might be confusing. If you are like me, you may think of it this way:

> If a class has more than one responsibility, you need to apply SRP onto the class.

This is what the definition says. Uncle Bob says each responsibility is an axis of change. So it is important to understand what a responsibility is. Now take a look at that.

## What Is a Responsibility?
Uncle Bob says a responsibility is “a reason for change” in the context of SRP. So we may write the definition of the SRP for explanation purpose only as:

![an image](https://raw.githubusercontent.com/unclexo/articles/main/assets/solid/the-understanding-of-single-responsibility-principle.png)

This means a class should have only one responsibility, not more. What if a class has more than one responsibility? What is the problem then? If a class has more than one responsibility, then the responsibilities become coupled. This means that changes to one responsibility can have a domino effect on the other classes. Therefore, one class should not depend on the way another works. If this is the case, then what about our class?

### What About Our Class?
We have seen three responsibilities in our class. This means the Contact class has more than one responsibility. Among them, the validation and the sending email are noteworthy. These break the SRP. These are reasons to change the class. That means we have two reasons to apply SRP onto our class. Why?

What if the validation API changes `notEmpty` validation rule to `required` to give it a more specific name?

Well, we have to go back to our `Contact` class and replace `notEmpty` with `required` in all places in the `validate()` method. That means changes to validation API affects the `Contact` class.

Likewise, if any changes happen in the mail API used in `sendMail()` method, that changes would make our `Contact` class change again.

This totally violates SRP.

### What is not a Responsibility?
If you think that each method of a class is a responsibility, then you are not on the right track. If so, you would get a class with constructor method only, each time you apply SRP onto your class. We could have had more methods in our Contact class. For example:

* `getData()` method to get user * provided data.
* `setData()` method to set user provided data.
* and more…

The `getData()` would have been an accessor method to return user data while `setData()` would have been a mutator method to set user data in `$this->data` attribute. These methods would not be SRP’s responsibilities.

It is perfect time to remember Cohesion here. Remember? If we had had these methods in our class, there would have been a strong relationship between these two methods and data. Why? Because these methods would deal with only data but not other contexts.

Think of how relevant are those methods in the context of data and class? For example, the context of our Contact class is all about preparing contact data (in object-oriented approach) but not validation or sending email. These are totally different contexts.

### Eventually!
I said we have three responsibilities in our Contact class. But now, we know what responsibility is. So the 1st responsibility is not really a responsibility in our class in the context of SRP. I just used it for demonstration purpose. But two other responsibilities are absolutely right. Any change to one of them makes the Contact class change. So we have to apply SRP onto our Contact class. Now take a look at how our contact app should look like:

```php
<?php

// Say, we would get data from a form submitted by a user
$data = [
  'firstName' => 'Jobaer',
  'lastName' => 'Sajeeb',
  'email' => 'unclexo@test.com',
  'message' => 'This is a test message'
];

// Prepares contact
$contact = new Contact($data);

// Prepares validator
$validator = new Validator();
$validator->setRules(/* set validation rules */);
$validator->setData($contact->getData());

// Prepares mail
$mail = new Mail();
$mail->to('business.owner@test.com', 'Contact App Owner');
$mail->from($contact->email, $contact->lastName);
$mail->subject('Contact App');
$mail->body($contact->subject);

// Validates data and sends email
if ($validator->validate()) {
  $mail->send();
}
```

Notice we separate validation and mail APIs from Contact class to the app script. Our Contact class does not depend on them anymore. The class is now quite independent. It has now only one responsibility that is preparing data: