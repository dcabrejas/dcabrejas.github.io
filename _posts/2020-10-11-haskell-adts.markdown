---
layout: post
title:  "What are Algebraic Data Types?"
date:   2020-10-11 10:41:11 +0100
categories: software-development haskell
tags: software-development haskell
---

Algebraic Data Types (ADTs) are becoming very popular recently. They are making it more and more into mainstream programming languages. As a programmer, you will come across the term sooner or later and I think it is important to understand what they are and have some theoretical background about them.

Some programmers might find the term abstract and misterious. Good news is that they turn out to be very simple things. In this post, we will explore what they are as well as some simple mathematics behind them. The idea is that we are able to count how many possible values a given type can hold.

There are some tiny pieces of code in the article with examples of type declarations. They are written in Haskell but no Haskell experience is required to read and understand them.

There are two types of ADTs, Sum types and Produt types.

## Combinatorics

We'll be looking at some combinatoric rules in order to understand properties for ADTs, so we'll give a brief introduction first:

Combinatorics is an area of mathematics concerned with counting. Turns out the idea of counting is why ADTs are calld sum and product types. We'll see how Combinatorics and ADTs are related in the next sections.
As I explain Sum and Product types, I'll first introduce the Combinatoric's rule associated with it before showing examples using types in a programming language.

## Sum Types

In Combinatorics, there is a basic rule called **The sum rule**.
The sum rule basically says that if you want to create something, and you can do so in $$ x $$ number of ways or in $$ y $$ number of ways, such that there is no way that belongs to both, x and y, then: The total number of ways to create that thing is $$x + y$$.

Example:

Suppose you want to cook a cake, you have two recipe books, one for English cakes and another one for French cakes. The English book has 3 recipes and the French book has 4. There is no recipe that appears in both books.

How many ways of making a cake do you have? 

$$ 3 + 4 = 7$$

Ok, so with that out of the way, let's look at an example of a Haskell Sum Type:

{% highlight haskell %}

data State = 
      Green
    | Yellow
    | Red

{% endhighlight %}

Ok, this models the state of a traffic light, it's simple but good enough for our pruposes. You might be thinking that this looks exactly like an Enum type in other languages and you would be correct.

By the way the text after the `=` sign, `Green | Yellow | Red` is called the data constructor in Haskell. We'll refer to it by that name from now on. The `|` character can be thought of as a logical **OR** symbol:  $$\lor$$

Right so the question is: How many ways do we have of constructing a `State`?

Well, we have 1 way of creating a `Green` value, 1 way of creating a `Yellow` value and 1 way of creating a `Red` value. Using the sum rule we learned above, we have $$1+1+1=3$$ possible ways to construct a value of type `State`.

Let's see a slighly more complex example: 

{% highlight haskell %}

data Optional a = 
    | None
    | Only a

{% endhighlight %}

This type is a reimplementation of Haskell's `Maybe` type. This type is
interesting because it's type polymorphic. `a` is a type variable and can be replaced with any type. For our pruposes we are going to be looking at the type 
`Optional Bool`.

So how many possible ways do we have of constructing a value of type `Optional Bool`?

Well, we have again 1 way of constructing a value of type `None`. However this time we have 2 ways of constructing a value of `Only Bool`, namely `Only False` and `Only True`. Therefore the total number of possible values is : $$2 + 1 = 3$$

## Product Types

Similarly to Sum types, there is a rule in Combinatorics for Product types. It is amusingly called **the product rule**. The product rule states that if you need to
make something that requires $$1,2,...n$$ steps, and each step can be done in 
$$x_1,x_2,...x_n $$ ways, then the total number of ways to make the thing is:

$$x_1 \cdot x_2 \cdot ... \cdot x_n$$

Example:

You want to elect a commitee in your company and you want to choose one person from each of the following departments $$ S = \textrm{Sales}, D = \textrm{Development}, CS = \textrm{Customer Services} $$

Development department has 4 employees,
Customer Services has 3 employees, and
Sales has 2 employees

None of the employees belong to more than one department.

How many ways of electing the commitee are there? 

$$ 4 * 3 * 2 = 24$$

This is because we have 4 ways of choosing a Developer, and **for each one** of those 4 ways, we have 3 ways of choosing a person from Customer Services, and similarly **for each one** of those, we have 2 ways of choosing a person from Sales.

Right, so let's see an example of Product types in Haskell:

{% highlight haskell %}

data TrafficLight = Bool State

{% endhighlight %}

This type models a traffic light, it has a Bool value which indicates whether the traffic light is operational or not and our State value we declared above to represent the current state of the traffic light (green, orange, red).

Right so this is called a product, as you probably noticed, it has no disjuction operator `|`. Instead, it has a space, which in the context of type constructors you can think of as the logical **AND** symbol $$\land$$. This means that in this case our TrafficLight type has **both**, a boolean value and a State value.

So in how many possible ways are there to construct a value of type `TrafficLight`?

Well, we have 2 possible ways of creating a `Bool` (True, False). When it comes to `State`, we saw above that we have 3 possible ways (Green, Yellow, Red).

We must first construct a Bool value and then construct a State value so we can use our product rule from combinatorics:

$$ 2 \cdot 3 = 6 $$

Let's list them out to see if this is true:

| Bool  | State  |
|-------|--------|
|  True |  Green |
|  True | Yellow |
|  True |   Red  |
| False |  Green |
| False | Yellow |
| False |   Red  |

You can imagine how this can be used to calculate more complex types, made up of Product and Sum types. It can get as complicated as you want, but we can always rely on these basic rules to count the possible number of values they can hold. That's pretty cool!

## Bonus (Function Types)

Finally, one cool thing we can do using the rules above is counting how many possible implementations a given function has. For example, let's suppose we have the following function:

{% highlight haskell %}

randomFunction :: State -> Bool
randomFunction = undefined

{% endhighlight %}

Can we know how many possible ways are there of implementing this function, just by looking at its type signature? Sure thing!

This function maps a value of type `State` into a value of type `Bool`, let's see in how many ways we can do that. For each value of type `State` we need to choose a value from `Bool` that it maps to. $$ \lvert \textrm{Bool} \lvert = 2 $$, so for each of `Green, Red, Yellow` we have 2 possibilities. The total number of ways to define this function is: 

$$ 2 \cdot 2 \cdot 2 = 2^3 = 8 $$

More generally if you have a function from set $$A$$ to set $$B$$, then there are:

$$ \lvert B \lvert^{\lvert A \lvert} $$ ways to implement that function. Using the product rule of combinatorics.
