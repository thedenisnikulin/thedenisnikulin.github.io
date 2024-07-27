+++
title = 'Type-level Bubble Sort in Rust: Part 1'
date = 2022-03-03T00:00:00+00:00
draft = false
+++

In this series of articles, we are going to implement bubble sort algorithm on the type level using (abusing) Rust's type system, which is [Turing-complete](https://sdleffler.github.io/RustTypeSystemTuringComplete/). 

The goal I want to accomplish by these articles is to help you understand what type level programming feels like, try to clear what's behind of all these "traits" and "impls", and to show what Rust's type system is capable of.
 

Before jumping in, you may need to have a basic understanding of Rust programming language (although the understanding of some FP language like Haskell or Scala should be also good enough).

Basically, type level programming allows us to carry the computations to the compilation phase where the compiler infers relationships between types, rather than computing the values during the program runtime.

![Some memes](/prin3njjdqf4r3zgiuaz.png)

## How do we express conditional logic with types?
We will use the power of **traits**. Traits are just like interfaces in C#/Java, except that you can implement traits for existing types (so the mindset is not "a type implements this interface" but rather "there's an implementation of this trait for a type").
Combined with generics, it gives us an ability to write multiple trait implementations for a single type just with different generic type arguments, allowing the compiler to **decide** what implementation must be picked for a particular case. Let's call it [multidispatch](http://smallcultfollowing.com/babysteps/blog/2014/09/30/multi-and-conditional-dispatch-in-traits/).


We will see this in practice a bit later, for now, let's focus on numbers.

## Numbers

Well, how do we represent type-level numbers? We can certainly declare a lot of structs for each possible natural number like `struct Num1; struct Num2; etc ` but that's just not a good idea (and is actually impossible since there are infinite amount of natural numbers). We will use [Peano number encoding](https://en.wikipedia.org/wiki/Peano_axioms). 

Simply put, natural numbers are just all non-negative numbers counted from 0 to infinity one by one. There comes the hint! One comes after zero, two comes after one, so we can say that number 1 is a successor of 0, and number 2 is a successor of successor of 0. This is how Peano numbers encoding work!

In Rust code it's going to look this way:
```rust
struct Zero;
struct Succ<N>(N);
```
For example, in order to represent number 4 we will write this:
```rust
Succ<Succ<Succ<Succ<Zero>>>>
```

## Number comparison

Next important thing about numbers in context of sorting is **comparison**.

Peano numbers comparison is based on the following rules:
1. `0 <= X` for every X
2. `Succ(X) >= Succ(Y)` if X >= Y

For example, let's try to calculate whether 2 is less than 3 with the help of the above rules. Is 2 less than 3? We can't certainly answer, and according to the 2nd rule we need to compare their predecessors. Is 1 less than 2? Again, we can't say. Is 0 less than 1? Yes, it is true according to the 1st rule. We have proved that 2 is less than 3.

Let's code it.

We will use traits and associated types. Look at this piece:
```rust
trait Compare<Rhs> {
    type Output;
}
```
This trait will be implemented for natural numbers.
Also, we will see multidispatch in action.

Let's write an implementation for the 1st rule of comparison:
```rust
// Some zero-sized types for representing equality
struct EQ;
struct LT;
struct GT;

// we've got to impls since we have no "less or equal to" type
impl Compare<Zero> for Zero {
    type Output = EQ;
}

impl<A> Compare<Succ<A>> for Zero {
    type Output = LT:
}
```
The meaning of these impls are `Comparing Zero with Zero produces EQ` and `Comparing Zero with Succ(A) produces LT`. As you can also see, the type for which the trait is implemented is used as _left hand side_ of comparison, and the type parameter `Rhs` is the _right hand side_.

In order to invoke the implementation, we need to write
```rust
<Zero as Compare<Zero>>::Output
```
which means "get an implementation of trait Compare<Zero> for Zero, and get the associated Output from it".

![Inferred type](/lxhqoutaxdr2c1bb5gl8.png)

As you can see, the compiler inferred the exact type that we needed. We have just made a computation on type level!  


As you have seen, the `<Zero as Compare<Zero>>::Output` thing is kind of awkward (imagine if there are even nested invocations), and we can actually simplify it with the type aliases:

```rust
type Cmp<Lhs, Rhs> = <Lhs as Compare<Rhs>>::Output;
```
It makes the code MUCH more readable, and allow us to look at traits as just **functions operating on types**. The generic type parameters (`Lhs, Rhs`) are function's parameters, associated types are what function returns (`Output`).

Notice, that I didn't write the implementation for comparison between `Succ<A>` and `Succ<B>`. Let it be your home assignment (you can see the solution in the code repository, the link is down below). A hint: it involves recursion.

Thank you for reading the article, that's all for today! In the next part of the series, I am going to discuss type-level lists. Please, leave comments below if you like the article and in case you spotted any mistakes or haven't understood something, I am open for critics and discussion!

The project's source code: 
>https://github.com/thedenisnikulin/type-level-sort
