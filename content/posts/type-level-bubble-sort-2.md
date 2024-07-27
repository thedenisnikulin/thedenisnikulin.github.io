+++
title = 'Type-level Bubble Sort in Rust: Part 2'
date = 2022-03-16T00:00:00+00:00
draft = false
+++

Hello everyone! In the [previous post from this series](https://dev.to/thedenisnikulin/type-level-bubble-sort-in-rust-part-1-3mcb) we discussed traits, type-level number representation, and implementation of basic type-level computations. The topic of this article is type-level lists. Make some tea or coffee and let's dive into the details!

![Rust-chan UwU](/cfu16rhpxitb5hlza9tv.jpeg) Image source: [link](https://twitter.com/maiRandomness/status/1011951419228852224?s=20&t=X-E0v8wIzSATDxXeNEmZZQ)

## Type-level lists

There's a data structure that you are most probably familiar with - a linked list. Each element of a linked list stores a value and points to the next element, and the last element of such list points to nothing. This data structure is easily represented with **recursion**, where the "nothing" is the base case, and an "element" is the recursive case. We actually will work with recursion quite a lot (the comparison of type-level numbers was also based on recursion), so you may need to learn to understand it.

The way we can define the list in Rust:
```rust
struct Nil; // a "null" ref
struct Cons<H, T>(H, T); // value and a ref to next element
```
The way we can use it in Rust:
```rust
Cons<Succ<Zero>, Cons<Zero, Cons<Zero, Nil>>> // a typical type-level list
Cons<Zero, Nil> // smol type-level list
Nil // the smolest of them all
```
The above examples are the same thing as `[1, 0, 0]`, `[0]`, and `[]` respectively on the value level.
The names `cons` and `nil` are terms from functional programming, but don't let them confuse you, it's just a fancy way of saying things ([wikipedia page](https://en.wikipedia.org/wiki/Cons) if you're curious). Also, the type parameters are named after terms `head` and `tail`.

## Defining basic computations
When implementing bubble sort, we will need a lot of helper "functions" for things like concatenation, swapping, and some others operations on list's elements. Let's define a trait that will represent an operation for `prepend`ing a number to a list.

```rust
// the naming "ComputeX" is used only for convenience,
// we will have a type alias named simply "Prepend" later on.
trait ComputePrepend<N> {
    type Output;
}
```
The number `N` will be prepended to the list for which this trait will be implemented.

The implementations:
```rust
impl<N> ComputePrepend<N> for Nil {
    type Output = Cons<N, Nil>;
}
```
This is quite simple: we prepend arbitrary `N` to `Nil` and that results in `Cons<N, Nil>`.

```rust
impl<N, H, T> ComputePrepend<N> for Cons<H, T> {
    type Output = Cons<N, Cons<H, T>>;
}
```
Here we prepend `N` to `Cons<H, T>`. This also seems not so hard to get.

Let's define the type alias:
```rust
type Prepend<N, L> = <L as ComputePrepend<N>>::Output;
```
and test it:

![prepending demo](/gnwjft9wgkix90yky87z.png)

The compiler inferred `Cons<Zero, Cons<Succ<Zero>, Nil>>` for us, great! Now we have a basic understanding of numbers, lists, and trait multidispatching. We can use it to write the rest of helper traits for our bubble sort algorithm. But first, let's define the algorithm itself.

## Recursive bubble sort algorithm

Here's the implementation of value-level bubble sort algorithm in pseudo-code with heavy usage of recursion.
```rust
fn bubble_sort(arr)
    if (arr.len() == 0) return arr
    bubbled = bubble(arr)
    return prepend(bubbled.head, bubble_sort(bubbled.tail));

fn bubble(arr)
    if (arr.len() == 0) return arr
    return prepend(arr.head, swapIfLess(arr.head, bubble(arr.tail)))

fn swapIfLess(head, tail) -> arr // swap tail[0] with head if tail[0] < head
fn prepend(head, tail) -> arr //...
```
Quick code explanation. In the function `bubble` , we move the least element of `arr` to the very beginning of the list (i.e. to the head). Then we cut the head off of the arr (since we consider the head sorted) and do the same `bubble` thing on the tail and again cut the head of that tail off, until the length is 0, and then we construct the whole array by prepending the heads back to their places.

Notice that the implementation is adapted so that it is more convenient for us to express the logic on traits. For example, instead of using `swapIfLess` it should be written as `if (...) swap(...)`. Although we could define some traits to define `if-else` logic, I find it cleaner for now to settle for `swapIfLess` thing. 

Actually, there's a problem with this approach. Look at this line:
```rust
return prepend(arr.head, swapIfLess(arr.head, bubble(arr.tail)))
```
The code assumes that `arr` is passed around by reference, like in javascript for example. For instance, if `swapIfLess(arr.head, ...)` function mutates `arr.head` as a result of swapping, the head from `prepend(arr.head, ...)` is also mutated. We can't mutate anything on the type level, so we need to adapt the implementation for this case.
To solve this, I will define conditional `swap` and `prepend` functions as one trait.
> It doesn't mean that there's no other workaround for this problem, if you have a better solution, I would be happy to see it!

## Bubble sort helper traits

Let's define that big-arse `SwapPrepend` trait:

```rust
trait ComputeSwapPrepend<E, H> {
	type Output;
}
```
Where `E` is for "equality" and `H` is for "head". We will define implementations of this trait for different equality types, so for each equality type the computation result will be different (i.e. a decision "to swap or not to swap" depending on the equality of `head` and `tail[0]`).

```rust 
// For Nil, we just prepend the `Head`
impl<E: Equality, Head: Nat> ComputeSwapPrepend<E, Head> for Nil {
    type Output = Cons<Head, Nil>;
}

// When `Head` and `A` (i.e. 'head' and 'tail[0]' as in the
// pseudo-code example above) are equal, also just prepend the `Head`
impl<A, Other, Head> ComputeSwapPrepend<EQ, Head> for Cons<A, Other> {
    type Output = Cons<Head, Cons<A, Other>>;
}

// When `Head` > `A`, just prepend
impl<A, Other, Head> ComputeSwapPrepend<GT, Head> for Cons<A, Other> {
    type Output = Cons<Head, Cons<A, Other>>;
}

// When `Head` < `A`, we finally swap and prepend.
impl<A, Other, Head> ComputeSwapPrepend<GT, Head> for Cons<A, Other> {
    type Output = Cons<A, Cons<Head, Other>>;
}

type SwapPrepend<E, H, L> = <L as ComputeSwapPrepend<E, H>>::Output;
```

Next, we define one more comparison trait, but this time it will compare **a natural number** with **a list** (i.e. with the list's head). It will help us to compare numbers when we will be implementing a trait for `Cons<Head, Tail>` where we can't simply get the first element of `Tail` to compare it with the `Head` with the old `Compare` trait, so the new `Compare` trait will help us with resolving the `Tail`'s first element.

```rust
trait ComputeCompare<Rhs: Nat> {
    type Output: Equality;
}

// This impl is only used for completeness.
// We will use `Compare` trait in tandem with `SwapPrepend` trait,
// and `<Nil as SwapPrepend<T, H>>::Output` thing doesn't 
// compare anything and only prepends.
// So the `Output` type doesn't matter in this context.
impl<Num: Nat> ComputeCompare<Num> for Nil {
    type Output = LT;
}

// Just compare `Head` with `Num`.
// `CompareNat` is the old natural number comparison trait.
impl<Head, Num, Tail> ComputeCompare<Num> for Cons<Head, Tail>
    where Head: Nat + ComputeCompareNat<Num>,
          Num: Nat + ComputeCompareNat<Head>,
          Tail: List
{
    type Output = CompareNat<Head, Num>;
}

type Compare<N, Ls> = <Ls as ComputeCompare<N>>::Output;
```
Above are also some trait bounds for the traits which we should specify to make the situation clear for the compiler.

They are pretty simple:
```rust
trait Nat {}
impl Nat for Zero {}
impl<N: Nat> Nat for Succ<N> {}

trait List {}
impl List for Nil {}
impl<H: Nat, T: List> for Cons<H, T> {}

trait Equality {}
impl Equality for EQ {}
impl Equality for LT {}
impl Equality for GT {}
```

And finally, some little things which will come in handy:

```rust
type HeadOf<L> = <L as ComputeHead>::Output;
type TailOf<L> = <L as ComputeTail>::Output;
```
Their functionality is quite simple, try to understand how they are implemented by yourself.


## Bubbling

Now, we are on the key part of the algorithm.

```rust
trait ComputeBubble {
    type Output;
}

impl ComputeBubble for Nil {
    type Output = Nil;
}

impl<Head, Tail> ComputeBubble for Cons<Head, Tail>
	// Some big & scary bounds
    where Head: Nat,
          Tail: List + ComputeBubble + ComputeCompare<Head>,
          <Tail as ComputeBubble>::Output: ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>
{
    type Output = SwapPrepend<Compare<Head, Tail>, Head, Bubble<Tail>>;
}

type Bubble<Ls> = <Ls as ComputeBubble>::Output;
```
The `Output` type computation is not that difficult, try to understand how it does the job by yourself.
You can see that the 3rd trait bound is quite ugly. It just means something like "The result of `bubbling the Tail` must support `swap & prepending the Head`".

## Bubble sorting

```rust
trait ComputeBubbleSort {
    type Bubbled: List;
    type Output: List;
}

impl ComputeBubbleSort for Nil {
    type Bubbled = Nil;
    type Output = Nil;
}

impl<Head, Tail> ComputeBubbleSort for Cons<Head, Tail>
	// Oh geez...
    where Head: Nat,
        Tail: List + ComputeBubble + ComputeCompare<Head> + ComputePrepend<Head> + ComputeBubbleSort,
        <Tail as ComputeBubble>::Output: ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>,
        <<Tail as ComputeBubble>::Output as ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>>::Output: ComputeHead,
        <<Tail as ComputeBubble>::Output as ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>>::Output: ComputeTail,
        <<<Tail as ComputeBubble>::Output as ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>>::Output as ComputeTail>::Output: ComputeBubbleSort,
        <<<<Tail as ComputeBubble>::Output as ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>>::Output as ComputeTail>::Output as ComputeBubbleSort>::Output: ComputePrepend<<<<Tail as ComputeBubble>::Output as ComputeSwapPrepend<<Tail as ComputeCompare<Head>>::Output, Head>>::Output as ComputeHead>::Output>
{
    type Bubbled = Bubble<Cons<Head, Tail>>;
    type Output = Prepend<HeadOf<Self::Bubbled>, BubbleSort<TailOf<Self::Bubbled>>>;
}
type BubbleSort<Ls> = <Ls as ComputeBubbleSort>::Output;
```
Again, the `Output` type is not what scares, but the trait bounds. Actually, I didn't write them by myself, these bounds where suggested by compiler. We can think of them as just a thing to make the compiler satisfied about the types. You can run `rustfmt` on the code to prettify it if you want to dig into what, for example, the last bound is for, but it can be briefly described as "the result of computing trait A must implement trait B", but following the whole chain of trait invocations.

As I know, there isn't a way to specify an implied trait bound (that is, we know that the result of `BubbleSort<TailOf<Bubble<Cons<..>>>>` is a `List`, but for the compiler to be satisfied we must write the whole chain of invocations like `<<Cons<..> as Bubble>::Output as TailOf>::Output as BubbleSort and etc` in the trait bounds) in current version of Rust, but there's [an RFC](https://rust-lang.github.io/rfcs/2089-implied-bounds.html) for this thing which is not implemented yet.

## Testing

![Bubble sort testing](/4aoi43ihq8hmoymm7oys.png)

The computed natural number types were expanded, but you can see that the list became sorted (the `N1`, `N2` types are just aliases for `Succ<Zero>` and `Succ<Succ<Zero>>`).

## Conclusion

The implementation is far from being perfect, but at least it works. There are ugly trait bounds which are completely unreadable, and I don't know any way to simplify them. But it's enough to show what the Rust's type system is capable of.
It is not worth to say that this variation of algorithm is not practically useful. But it is a good way to challenge your mind and try something extraordinary! 

The full source code:
> https://github.com/thedenisnikulin/type-level-sort

## Links

[A much more practically useful type-level programming in Rust](https://willcrichton.net/notes/type-level-programming/) \
[A macro to define type-level logic with value-level syntax in Rust](https://github.com/willcrichton/tyrade) \
[Type-level Brainfuck in Rust](https://sdleffler.github.io/RustTypeSystemTuringComplete/) \
["Gentle Intro to Type-level Recursion in Rust" ](https://beachape.com/blog/2017/03/12/gentle-intro-to-type-level-recursion-in-Rust-from-zero-to-frunk-hlist-sculpting/) \
[Type-level registers in Rust](https://blog.auxon.io/2019/10/25/type-level-registers/) \
[Type-level quicksort in Scala"](https://blog.rockthejvm.com/type-level-quicksort/) \
[Type-level sorting algorithms in Haskell](https://gist.github.com/pxqr/3754181) \
[A repo with functions and algorithms implemented purely on types in TypeScript](https://github.com/ronami/meta-typing) \
