---
layout: post
title:  "Thoughts on Rust from a PHP developer!"
date:   2018-08-27 15:41:11 +0100
categories: software-development rust
tags: software-development rust
---

I first starting learning Rust about 6 months ago, I was looking for a new language to learn when I came across it.
At first I thought Rust was only meant to be a low level, systems programming language, but the more I learned, the more I realised the potential it has for
high level programming and web applications. Also, along the way I learned many ways in which Rust prevents many of the typical bugs often found in applications written in other programming languages.

Let's explore some of the benefits you get by writing your web applications in Rust.

## Strongly Typed

In Rust, everything is strongly typed and checked at compile time.
If a function in your program accepts a parameter of type String for example and your program compiles successfully, you are guaranteed that the value
your function takes will always be a string. Therefore you can avoid writing defensive code that checks whether the parameter is of the correct
type at runtime or whether the parameter is null, this makes the program faster, removes the possibility of type related errors and makes you write less code.

{% highlight rust %}
pub fn generate_greeting(name: String) -> String
{
  format!("Hi {}!", name)
}

pub fn main()
{
  let greeting = generate_greeting(String::from("Diego"));
  println!("{}", greeting);    
}
{% endhighlight %}

It goes even further though, it also enforces the type of a value used in expressions such as `if` statements for example.
In other languages you can often use a non-boolean type such as a integer or a string as the condition of an `if` statement. This often leads to unexpected behaviour, with Rust however you must always provide a `boolean` type in this case, which ensures the result is what the programmer intended, Rust won't try to guess whether something is false or true based on some internal rules the programmer might or might not be familiar with, the following code will just not compile in Rust :

{% highlight rust %}
pub fn main()
{
    let number = 0;

    if number {
        println!("number is not falsy!");
    }
}
{% endhighlight %}

The compiler would show the following rather helpful error :

{% highlight rust %}
6 |     if number {
  |        ^^^^^^ expected bool, found integral variable
{% endhighlight %}


## Immutable By Default

In my view, another benefit of writing in Rust is that, by default, everything is immutable. This forces you to be explicit whenever you want to
allow something to be mutable, this protects from surprises where data gets mutated when you didn't intend it to.

Consider this example below :

{% highlight rust %}
pub fn walk_dogs(dogs: &Vec<&str>)
{
    for dog in dogs {
        println!("Walking a {}...", dog);
    }
}

pub fn main()
{
    let dogs_collection = vec!["German Spitz", "Golden Retriever", "Akita"];
    walk_dogs(&dogs_collection);

    //this will always be true, because the dogs_collection variable is immutable
    assert_eq!(dogs_collection, vec!["German Spitz", "Golden Retriever", "Akita"]);
}
{% endhighlight %}

The fact that we can pass our collection of dogs into a method and be guaranteed that the data hasn't changed at all is quite mind-blowing and became an addictive feature of Rust to me. In the example above is obvious that the `walk_dogs` method doesn't mutate our collection but what if it was a method from another library? Will we then need to make sure the data hasn't changed after the function call? Even if the method doesn't change the value today, it may do in a future version of the library and we would not know about it.

With Rust on the other hand, we can rest assured the method won't mutate the data because we are passing what is called an immutable reference to the `dogs_collection` vector, if the `walk_dogs` method tried to mutate the data in any way, this code would not compile because it would be breaking Rust ownership rules. Ownership rules is one of the things that make Rust unique, you can read more about them in the [Rust book][rust-ownership]

If however we wanted the function to mutate the data, both the function signature and the code calling the function must be explicit about the intention of mutating the data by passing and accepting a mutable reference explicitly as in the following example :

{% highlight rust %}
pub struct Product
{
    name: String,
    price: f32,
    stock: i32
}


pub fn purchase_product(product: &mut Product, qty: i32)
{
    //purchase logic omitted
    product.stock = product.stock - qty;
}

pub fn main()
{
    let mut product = Product { name: String::from("Green Armchair"), price: 150.50, stock: 30};
    purchase_product(&mut product, 5);

    assert_eq!(product.stock, 25);
}
{% endhighlight %}

Here you can see that we immediately define the variable product to be mutable using the `mut` keyword, this tells Rust that we intend to mutate this variable, this introduces limitations such as that we can only have one mutable reference to a value at a time, which protects your code from data races for example. Notice that the `purchase_product` function signature must also explicitly specify that it accepts a mutable reference to a Product using the `&mut Product` syntax, the compiler makes sure this is the case. Finally we can see that the `product.stock` property is mutated inside the function, the quantity ordered has been subtracted and the stock available for the product object is now 25.

## Enums And Pattern Matching

Enums, or algebraic data types are used a lot in Rust. Let's focus on one example to illustrate the power of combining enums with exhaustive pattern matching.
One of my favourite design decision of Rust is that it doesn't have a `null` type. A variable in Rust can never be `null`. How can you express the absence of a value then? You guessed it, Enums!

{% highlight rust %}
pub enum Option<T> {
    None,
    Some(T),
}
{% endhighlight %}

`Option` is the de facto way of expressing the absence or the presence of a value in Rust, notice that `Option` is generic, `T` can be any type.
`Option` can be `None`, if it has no value or `Some`, if it does.

Let's see an example of some code making use of the Option enum :

{% highlight rust %}
pub struct Customer {
    id: i32,
    name: String
}

pub fn get_customer_by_id(id: i32) -> Option<Customer>
{
    //fetches customer from a DB
}

pub fn main()
{
    let customer = get_customer_by_id(32);
}
{% endhighlight %}

Here we have a struct `Customer`, think of it like a class in other languages such as PHP and a function which loads a customer from a DB.
The customer with the given id might not exists in the DB, therefore we need to return an Option to allow for the possibility a non-existing customer.
Now, we can't just assume that the customer returned by `get_customer_by_id` is an instance of Customer, if we tried to do that, our program would not compile,
the compiler not only won't allow you to assume you got an instance of Customer, it will also force you to exhaustively handle every possible variant of the enum returned by the function, making your code really robust.

{% highlight rust %}

pub struct Customer {
    id: i32,
    name: String
}

pub fn get_customer_by_id(id: i32) -> Option<Customer>
{
    Some(Customer {id: 32, name: "Diego".to_string()})
}

pub fn main()
{
    let customer = get_customer_by_id(32);

    match customer {
        Some(a) => println!("Customer name: {}", a.name),
        None => println!("Customer not found")
    }
}
{% endhighlight %}

Using the match operator, we are able to handle every possible outcome of calling the `get_customer_by_id` function.
Notice that we can extract data from enums, in this example the `Customer` instance ends up in the variable `a` in the first arm of the match expression, which we can then use on the right hand side of the `=>`.

Most programming languages don't force to exhaustively check the values of variables, lots of bugs come from assuming a function returned one type of value when sometimes does return another, Rust type system has your back and the compiler will not let you compile your code if you don't check all the possible types.

## Many More!

I have only scratched the surface here, there are many more aspects of Rust which helps you write faster and safer applications.
These include:

- Dependency management out of the box.
- An incredibly well-designed module system.
- No inheritance!
- Traits
- Fearless concurrency
- Etc

If you liked what you saw, and want to get started learning Rust, you can check out the official [Rust Book][rust-book].
Also you can see web related libraries and frameworks listed at [Are we web yet?][web-yet]

[rust-ownership]: https://doc.rust-lang.org/stable/book/2018-edition/ch04-00-understanding-ownership.html
[rust-book]: https://doc.rust-lang.org/stable/book/2018-edition/
[web-yet]: https://www.arewewebyet.org/
