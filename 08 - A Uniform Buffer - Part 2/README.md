# A Uniform Buffer (Part 2)

In the previous exercise, we declared a uniform buffer.  That was rather useless in and of itself,
so let's fix that.  In this exercise, we will read from that uniform buffer and finally initialize
`crucial_data` with data worthy of the name.

If you have been impatient and going to `solution.spvasm` prematurely before, try to really do this
one on your own.

## Type Pointers and Access Chains

This is where we can finally understand the point of "type pointers" (no pun intended, though if you
enjoyed it I'll let it slide this time).  Recall that variables are declared (with `OpVariable`)
as a type pointer (declared with `OpTypePointer`).  Later, load and store (`OpLoad` and `OpStore`
respectively) operate on these variables.  Truth is, load and store operate on any pointers (of
which variables are one example).

But what other pointers are there?

Let's take another point of view.  Say you have the following:

```c
struct points
{
    vec4 point[10];
};

uniform properties
{
    points geometry[3];
} props;
```

And you want to access:

```c
props.geometry[2].point[5].y
```

One way of reading this would be to `OpLoad` the whole buffer, then use various instructions to get
to the data we are looking for.  If this was writable (because it's not a uniform buffer, but a
storage buffer), that would get more tricky (read the whole thing, modify one part, then write it
all back), or impossible (with variable sized arrays).

SPIR-V instead has a nifty instruction called `OpAccessChain`.  `OpAccessChain` takes a pointer,
applies a number of indices in sequence and creates a new pointer.  The extra nice part is that this
"indexing" applies to all of:

- Selecting a member in a struct,
- Selecting an element in an array,
- Selecting a component of a vector, and
- Selecting a column of a matrix.

In other words, the whole expression above requires a single `OpAccessChain` which takes `props` and
produces `props.geometry[2].point[5].y`.  In this case, the indices used are `0 2 0 5 1`, which in
order are:

- Member 0 of `props` (i.e. `.geometry`), then
- Element 2 of that (i.e. `[2]`), then
- Member 0 of that (i.e. `.point`), then
- Element 5 of that (i.e. `[5]`), then
- Component 1 of that (i.e. `.y`).

> Waiting for you to review the above and make sure you completely understand it ...

## Accessing the Uniform Buffer

Let's get to work then.  We want to add the following line to `main`:

```c
crucial_data = props.smoothness;
```

First, we need to declare a constant for the index (which selects the member `smoothness`).  This is
not a `float` constants as we have so far seen, but rather `int`.  Zeroth, then, we should declare
the `int` type.  That is done with the `OpTypeInt` instruction.

`OpTypeInt` takes a _Signedness_ parameter.  Do we want to declare these constants as signed or
unsigned integer?  The truth is, it doesn't matter.  _Signedness_ is largely unused (helps mostly
with validation and checking), and the instructions themselves dictate whether an integer is treated
as signed or unsigned.  Nevertheless, let's go with signed; `OpAccessChain` says it treats these
indices as signed.

You have already learned how to declare constants, so declare one with a value of 1 (because
`smoothness` is member 1 of the uniform buffer struct type).  As a reminder, `OpConstant` is the
instruction to use here.

Next, we need to use the `OpAccessChain` instruction.  This instruction goes in `main`, for example
right after the `OpStore` instruction.  Look it up now and see if we have everything we need for it.

> Waiting for you to consult the spec ...

What should we use as the _Result Type_?  What is the type of `props.smoothness`?  If you don't see
a type pointer declaration (an `OpTypePointer` instruction that is) for this type, go ahead and add
one!

> Waiting for you to come up with the `OpAccessChain` instruction ...

Now that we have a pointer to what we need to load, it's business as usual.  Let's load (`OpLoad`)
from this shiny new pointer and store (`OpStore`) it in the pointer for `crucial_data`.  Again,
these go in `main`, right after the `OpAccessChain` instruction.

As always, use `./validate` to validate and see what GLSL you end up with.

> Waiting for you to complete this part of the exercise ...

Stuck?  Don't give up!  Why not reread this whole exercise?

> Waiting for you to really try to complete this part of the exercise without looking at
> `solution.spvasm` ...

Good job!  This was probably the second most exciting part of SPIR-V.  By the way, now that our
little SPIR-V is not so little anymore, you could see a little summary of what each exercise does by
taking the `diff` of the two files:

```bash
$ diff exercise.spvasm solution.spvasm
```

## (Optional) The Extra 1.6Km

If you like to experiment more with `OpAccessChain` and type declarations, feel free to experiment
and construct more complicated types, for example with arrays (`OpTypeArray`), matrices
(`OpTypeMatrix`) and nested structs (`OpTypeStruct`).
